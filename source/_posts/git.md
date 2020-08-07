---
title: git的安装与使用
date: 2020-08-02 22:26:22
tags:
- git
- tutorial
categories:
- [版本控制（git svn）]
---

本文介绍git在ubuntu上的安装与使用。

<!-- more -->

## git安装与配置 
ubuntu系统不自带git，需要自己安装：

``` bash
sudo apt-get install git
```

安装完成后，设置用户名和邮箱地址：

``` bash
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR EMAIL ADDRESS"
```

## 创建本地版本仓库

在库文件夹比如learngit下初始化git仓库：

``` bash
git init
```

在文件夹中添加文件，比如：classification.cpp，然后可以添加文件到git仓库，分两步进行：
第一，用git add命令告诉git，将文件添加到仓库：

``` bash
git add README.md
```

第二，用git commit命令告诉git，将文件提交到仓库：

``` bash
git commit -m "add a new file"
```

## 添加到远程仓库

git是分布式版本控制系统，同一个git仓库可以分布到不同的机器上，这儿的机器，可以指公司的git服务器或者github，下面以github为例，介绍怎样将本地git仓库添加到远程仓库。

本地仓库和github的连接有两种方式：
 * HTTPS连接（比如地址为：`https://github.com/gitusername/learngit.git`）
 * SSH连接（比如地址为：`git@github.com:gitusername/learngit.git`）

### HTTPS连接

使用HTTPS连接的不同之处在于：其一，不需要SSH key；其二，上面第一条命令地址要改为HTTP地址：

``` bash
git remote add origin https://github.com/gitusername/learngit.git
```

步骤简单，也是github推荐的连接方式，然而其缺点为速度比SSH连接方式慢。

### SSH连接

SSH加密传输首先要有SSH key，生成过程包括：
第一步，创建SSH key。输入下面指令：

``` bash
ssh-keygen -t rsa -C "youremail@example.com"
```

之后可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

第二步，登陆GitHub，打开“Account settings”，“SSH Keys”页面，然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容。

然后，在github上创建一个名为learngit的空的仓库（不包含README文件）。

将本地仓库与github仓库相关联并推送内容包含下面两步：
第一，输入下面命令：

``` bash
git remote add origin git@github.com:gitusername/learngit.git
```

其中，origin代表远程库，是git的默认叫法。

第二，将本地库内容推送到远程库：

``` bash
git push -u origin master
```

由于远程库是空的，第一次推送master分支时，加上了-u参数，git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。再次推送到远程github仓库时，输入简化的命令即可：

``` bash
git push origin master
```

## 从远程仓库克隆

上一节介绍先有本地库，后有远程库的时候，如何关联远程库。假设我们从零开发，最好的方式是先创建远程库，然后，从远程库克隆到本地。

首先在github上创建一个新的仓库，仓库名为learngit，这次要勾选Initialize this repository with a README。

远程库已经准备好了，下一步是用命令git clone克隆一个本地库，也分两种方式。
SSH连接克隆命令为：

``` bash
git clone git@github.com:gitusername/learngit.git
```

HTTP连接克隆命令为：

``` bash
git clone https://github.com/gitusername/learngit.git
```

在本地仓库修改完代码后，推送到github库：

``` bash
git push origin master
```

---

## git常用命令

### 常规操作

``` bash
git status # 查看版本库状态

git log # 查看提交历史

git diff #　比较当前工作目录与版本库的差别

git add file　# 向版本库添加文件

git add directory/ # 向版本库添加文件夹

git commit -m "description" # 提交并添加描述

```

### 删除文件(夹)

``` bash
git rm file # 同时删除本地文件与远程分支文件
git rm --cached file # 只删除远程分支文件，保留本地文件

git rm -r directory #　同时删除本地文件夹与远程分支文件夹
git rm -r --cached directory # 只删除远程分支文件夹，保留本地文件夹
```

### 恢复到指定版本
项目跟踪工具的一个重要任务之一，就是使我们能够随时恢复到某一阶段的工作。

命令形式：

``` bash
git reset [--mixed | --soft | --hard]
```

其中：
　`--mixed` 仅是重置索引的位置，而不改变你的工作树中的任何东西，并且提示什么内容还没有被更新。这个是默认的选项。
　`--soft` 既不触动索引的位置，也不改变工作树中的任何内容，这个选项使你可以将已经提交的东西重新逆转至＂已更新但未提交（Updated but not Check in）＂的状态。
　`--hard` 将工作树中的内容和头索引都切换至指定的版本位置中，也就是说自 之后的所有的跟踪内容和工作树中的内容都会全部丢失。慎用。

举例：
撤销上次提交（commit），保留当前所有更改：

``` bash
git reset --soft
```

彻底恢复到上次提交的版本，放弃当前所有更改：

``` bash
git reset --hard
```

彻底返回到最近两次提交之前的版本：

``` bash
git reset --hard HEAD~2
```

### 管理分支

如果项目存在多个分支就需要进行分支管理。

查看分支:

``` bash
git branch
```

使用以下命令创建分支并将创建的分支设置为当前工作分支：

``` bash
git branch new_branch
git checkout new_branch
或者
git checkout -b new_branch
```

删除分支：

``` bash
git branch -d new_branch # 先检查分支是否合并到其他分支上，若没有合并则无法删除
git branch -D new_branch # 直接删除分支，不会检查分支状态
```

查看版本库的发展记录:

``` bash
git show-branch
```

查看两个版本的差异情况：

``` bash
git diff B1 B2
```

合并其他分支到主分支上：

``` bash
git check master
git merge -m "merge from new_branch" new_branch
或者
git pull . new_branch
```

合并远程库到本地主分支：

``` bash
git pull --rebase
```

**注意：** 
* 执行 `git pull --rebase` 的时候必须保持本地目录干净。即：不能存在状态为 modified 的文件（存在Untracked files是没关系的）。
* 如果出现冲突，可以选择手动解决冲突后继续 rebase，也可以用 `git rebase --abort` 放弃本次 rebase。

最后，将本地提交好的分支推送到远程仓库：

``` bash
git push origin master
```

---

## 参考
1. [git和github在ubuntu上的使用](https://blog.csdn.net/u012526120/article/details/49401871)
2. [git回到指定版本命令](https://blog.csdn.net/pcyph/article/details/44035935)
3. [git pull --rebase的正确使用](https://juejin.im/post/6844903895160881166)

