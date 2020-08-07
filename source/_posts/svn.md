---
title: svn的安裝与使用
date: 2020-08-03 23:55:05
tags:
- svn
- tutorial
categories:
- [版本控制（git svn）]
---

本文介绍如何在ubuntu上的搭建svn服务器以及如何使用svn。

<!-- more -->


## 安装svn服务器软件subversion
ubuntu系统不自带svn，需要自己安装：

``` bash
sudo apt-get install subversion
```

## 创建svn目录
在home下创建svn文件夹：

``` bash
sudo mkdir /home/svn
```

在新创建的svn文件夹下创建repo文件夹（svn的版本仓库存放目录）：

``` bash
sudo mkdir /home/svn/repo
```

## 创建svn版本仓库

``` bash
sudo svnadmin create /home/svn/repo
```

## 配置svn服务器

创建svn用户组：

``` bash
sudo addgroup subversion
```

将自己（比如：peng）加入到svn用户组：

``` bash
sudo usermod -a -G subversion peng
```

更改权限：

``` bash
sudo chown -R peng:subversion /home/svn/repo
sudo chmod -R g+rws /home/svn/repo
```

## 访问svn服务器
访问svn服务器有以下几种协议，下面会一一介绍：
<table>
  <tr>
    <th>协议</th>
    <th>方式</th>
  </tr>
  <tr>
    <td>file://　</td>
    <td>直接访问（在相同主机上)</td>
  </tr>
  <tr>
    <td>svn://</td>
    <td>通过svn用户协议访问</td>
  </tr>
  <tr>
    <td>svn+ssh://</td>
    <td>和svn://相同，只是通过ssh隧道</td>
  </tr>
  <tr>
    <td>http:// </td>
    <td>通过WebDAV协议访问subversion支持的Apache 2 web服务器</td>
  </tr>
　　<tr>
    <td>https://</td>
    <td>和http://次相同，只是用了SSL加密</td>
  </tr>
</table>

### file://

不需要做任何设置，直接通过下方指令即可获取本地svn服务器仓库中的资源：

``` bash
svn co file:///home/svn/repo
或者
svn co file://localhost/home/svn/repo
```

### svn://

需要配置/home/svn/repo/conf目录中的文件。

1. svnserve.conf: 服务配置文件：

``` bash
#匿名用户不可读
anon-access = none
#权限用户可写
auth-access = write
#密码文件为passwd
password-db = passwd
#权限文件为authz
authz-db = authz
```

2. authz: 用户权限配置文件：

``` bash
[groups]
subversion = peng # subversion组的用户

[/] # 必须写/，因为这表示从仓库的目录开始设置权限
@subversion = rw # subversion组有rw(读写权限)
* = r # 所有人有r(读权限)
```

3. passwd: 用户密码配置文件：

``` bash
[users]
 # harry = harryssecret
 # sally = sallyssecret
peng = 12345 # 设定用户peng的密码是12345，是的没错，密码是明文的。
```

启动svn服务器：

``` bash
svnserve -d -r /home/svn/repo
```
(-d: 表示在守护模式运行，-r: 指定服务器的根目录)
默认端口为3690，可以根据需要自行更改或者做些其它设置，详情请见`svnserve --help`。

停止svn服务器：

``` bash
killall svnserve
```

开机默认启动：

可以添加一个自动启动脚本`/etc/init.d/subversion`，设置 svn 服务开机默认启动。

``` bash
#!/bin/sh

# start/stop subversion daemon

test -f /usr/bin/svnserve || exit 0

OPTIONS="-d -T -r /home/svn/repo"

case "$1" in
 start)
  echo -n "Starting subversion daemon:"
  echo -n " svnserve"
  start-stop-daemon --start --quiet --oknodo --chuid root:root --exec /usr/bin/svnserve -- $OPTIONS
  echo "."
  ;;

 stop)
  echo -n "Stopping subversion daemon:"
  echo -n " svnserve"
  start-stop-daemon --stop --quiet --oknodo --exec /usr/bin/svnserve
  echo "."
  ;;

 reload)
  ;;

 force-reload)
  $0 restart
  ;;

 restart)
  $0 stop
  $0 start
  ;;

 *)
  echo "Usage: /etc/init.d/subversion (start|stop|reload|restart)"
  exit 1
  ;;

esac

exit 0
```

添加执行权限：

``` bash
sudo chmod u+x /etc/init.d/subversion
```

测试一下从脚本启动：

``` bash
sudo /etc/init.d/subversion start
```

将此脚本设置为开机默认启动：

``` bash
sudo update-rc.d -f subversion defaults
```

在svnserve已开启的情况下，通过下方指令获取svn服务器仓库中的资源：

``` bash
svn co svn://hostname repo --username peng
```

### svn+ssh://

设置同svn://，在ssh服务已经打开的情况下，通过下方指令即可获取svn服务器仓库中的资源：

``` bash
svn co svn+ssh://hostname/home/svn/repo repo --username peng
```

注意：这里必须加上仓库的完整路径（/home/svn/repo）, 而svn://方式则不用。


### http://

需要安装并配置Apache2 web server。

安装Apache2：

``` bash
sudo apt-get install apache2
```

配置Apache2：

在`etc/apache2/mods_available/dav_svn.conf`文件中添加以下代码：

``` bash
<Location /home/svn/repo>
    DAV svn
    SVNPath /home/svn/repo
    AuthType Basic
    AuthName "subversion repository"
    AuthUserFile /etc/subversion/passwd
    Require valid-user
</Location>
```

添加Apache2用户www-data并加入到subversion组中：

``` bash
sudo adduser www-data subversion
```

重启Apache2服务：

``` bash
sudo /etc/init.d/apache2 restart
```

创建用户和密码文件（`/etc/subversion/passwd`）：

``` bash
sudo htpasswd -c /etc/subversion/passwd peng
```
按照提示输入密码即可。
通过下面的命令可添加新用户（new_user）：
``` bash
sudo htpasswd /etc/subversion/passwd new_user
```

配置完成，通过下方指令即可获取svn服务器仓库中的资源：

``` bash
svn co http://hostname/home/svn/repo repo --username peng
```

### https://

设置同http://，但是需要为Apache2配置支持https的数字证书（略）。

然后通过下方指令即可获取svn服务器仓库中的资源：

``` bash
svn co https://hostname/home/svn/repo repo --username peng
```

## RabbitVCS

RabbitVCS是Linux平台下版本控制程序Subversion的GUI前端客户端，使用Python构建，可以与文件管理器Nautilus紧密整合，支持 Subversion(SVN), Git。可替代Windows下的TortoiseSVN。

Ubuntu用户安装:
``` bash
sudo add-apt-repository ppa:rabbitvcs/ppa
sudo apt-get update
sudo apt-get install rabbitvcs-core rabbitvcs-cli rabbitvcs-nautilus rabbitvcs-gedit
```

最后输入以下命令重启Nautilus就可以使用RabbitVCS了。

``` bash
nautilus -q
nautilus
```

---

## svn常用命令

以下访问服务器方式均以file:///home/svn/repo为例。

**`import`**: 将项目上传到svn服务器，跟commit对应，是将未版本化的文件导入版本库中的最快方法，它会根据需要创建中介目录。

``` bash
svn import myrepo file:///home/svn/repo -m "Initial import"
```

**`export`**: 将项目从服务器导出到本地，跟checkout对应，但是导出的文件夹不含.svn目录，脱离svn版本控制。

``` bash
svn export -r version file:///home/svn/repo
```
（通过 *`-r version`* 导出指定版本，不加则默认导出最新版本。）

**`checkout`**: 从服务器仓库下载项目到本地，成为本地工作副本。
``` bash
svn checkout file:///home/svn/repo -r version
或者
svn co -r version file:///home/svn/repo
```
（通过 *`-r version`* 导出指定版本，不加则默认下载最新版本。）

**`trunk, branches, tags`** 三大目录：

　trunk: 主干，一般把项目提交到此文件夹里面,在trunk中开发。
　branches: 分支，一般把那些需要打分支,但是有可能会修改的项目代码，打分支到此目录。
　tags: 分支，一般把那些阶段性(如迭代各期)的项目代码,打分支到此目录。

新建的svn仓库中没有这三个文件夹，可通过下面两种方式创建：

1. 将项目从服务器下载到本地，进入生成的项目目录执行以下代码：

``` bash
svn mkdir trunk tags branches
svn commit -m　"Creating trunk, tags, branches"
```

2. 如果不想将项目整个下载到本地，也可以直接使用一条命令创建并提交目录到svn服务器：

``` bash
svn mkdir file:///home/svn/repo/trunk -m "Creating trunk dir"
```

另外两个目录使用同样的方式进行创建并提交。

**`status`**: 查看工作副本状态。

``` bash
svn status
```

第一列表示文件的状态：
　，没有修订
　A，添加
　C，冲突，需要解决冲突状态，才能正常提交代码
　D，删除
　I，忽略
　M，有修改
　?，没有版本控制，在工作副本添加文件或目录之后，需要使用svn add your_path才能加该文件加到版本控制
　!，文件丢失，如果不是使用svn delete删除文件或目录，会产生此状态

**`info`**: 查看工作副本信息。

``` bash
svn info
```
能够查看到本工作副本的url、版本等信息。

**`update`**: 升级到新版本。

``` bash
svn update -r verison
```

**`add`**: 添加新文件或者目录到版本控制。

``` bash
svn add file
svn add dir
```

**`delete`**: 删除文件或目录。

``` bash
svn delete file
svn delete dir
```
如果仅仅是手动使用rm命令或窗口下删除工作副本内的文件或目录，该删除并不会记录svn的状态。可能会导致提交代码时，遗漏了删除文件或目录。因此建议删除svn工作副本内的文件或目录时，使用本命令进行操作。

**`commit`**: 提交代码。

``` bash
svn commit [-m message] [file_list]
```
如果没有带文件列表，默认把工作副本的所有修订都提交，如果有带文件列表，则只提交文件列表中对应文件的修订。

**`merge`**: 合并代码。

``` bash
svn merge -r ver1:ver2 src_url working_copy_path
```
可将任意版本的任意修订合并到工作副本中。如果ver1小于ver2，表示合并src_url分支ver1到ver2的修订到本地工作副本；如果ver1大于ver2，表示回退修订。

**`revert`**: 回退工作副本的修订。
``` bash
svn revert file
svn revert -R dir
```

**`log`**: 查看版本log。

``` bash
svn log [OPTIONS] [FILE_LIST]
```
默认只查看工作副本及以前版本的log，可以使用参数过滤得到我们想要的内容，详情请见 *`svn log --help`* 。

---

## 参考

1. [Subversion](https://help.ubuntu.com/community/Subversion)
2. [SVN基础用法](https://www.jianshu.com/p/0a8cf8bcdacc)
3. [Ubuntu搭建svn服务器](https://www.jianshu.com/p/6b83b6f2cace)
4. [Linux搭建SVN服务器，并设置开机默认启动](http://ourjs.com/detail/5b1ca77b7ad90c6e47f34b72)
