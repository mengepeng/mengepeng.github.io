---
title: Hello Hexo
date: 2020-07-29 17:55:05
tags: 
- hexo
- tutorial
categories:
- [hexo]
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

<!-- more -->

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

---

## 多设备使用Hexo

### 通过原电脑创建hexo分支

```bash
// git初始化
git init
// 新建分支并切换到新建的分支
git checkout -b 分支名
// 添加所有本地文件到git
git add .
// git提交
git commit -m "提交说明"
// 文件推送到hexo分支
git push origin hexo
```

### 在新电脑上搭建hexo博客

```bash
// 克隆分支到本地
git clone -b hexo https://github.com/mengepeng/mengepeng.github.io.git
// 进入博客文件夹
cd mengepeng.github.io
// 安装依赖
npm install
```

### 编辑并更新博客

每次编辑之前先拉取Github上hexo分支的源文件到本地，进行合并
```bash
git pull origin hexo
```

编辑之后把静态文件提交到主分支master上，源文件提交到hexo分支。
```bash
// 博文提交到master分支
hexo clean && hexo g && hexo d
// 源文件提交到hexo分支
// 添加源文件
git add .
// git提交
git commit -m "update info"
// push源文件到Github的hexo分支
git push origin hexo
```
