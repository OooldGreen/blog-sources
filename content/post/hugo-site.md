---
title: "关于配置博客"
date: 2022-10-17T10:11:51+08:00
lastmod: 2022-10-17T10:11:51+08:00
draft: false
keywords: ["Hugo", "jane"]
description: ""
tags: ["Jane"]
categories: ["Blog"]
author: ""

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---

<!--more-->
# Hugo搭建
## 下载Hugo
参考[官方文档](https://gohugo.io/getting-started/installing/)
### 1. 二进制（MACOS）
```
1 进入[Hugo Release](https://github.com/gohugoio/hugo/releases)
2 下载最新版本，解压缩（记得要下载extend版本）
3 把hugo文件放进/usr/local/bin
4 输入hugo version检查版本号，出现版本号，安装成功
```
### 2. Homebrew
或者更简单，如果你有homebrew，直接：
```
brew install hugo
```
## 生成我的第一支博客
在我的博客文件夹下输入以下代码：
``` markdown
hugo new post/my-first-post.md
```

Tips:
- 我选择的主题是[jane](https://github.com/xianmin/hugo-theme-jane)，配置的博客目录在post
- 名字一定要加后缀！起名字最好就是英文小写+短横线～

## 部署到Github
### 准备工作
#### 先有一个github账号
1. 进入[github](https://github.com/)（此处可能需要🪜）
2. 输入邮箱，点击Sign up for GitHub创建账号
3. 然后登录就好啦
#### 创建新的repository
> 一共需要创建两个仓库：
>   一个网站仓库，名字必须是`xxxx.github.io`
>   一个存放博客源码的仓库，我的参考[necolas](https://blog.nekolas.cafe/posts/hugo/hugo-deploy-your-blog/)的博客叫做`Hugo Sources` 

##### 先建一个仓库
```
1 点击右上角加号`+` -> `new repository`
2 输入你的`repository name`，必须是`xxxx.github.io`，例如我的仓库`snowcabin.gihub.io`
3 记得检查一下是否选择`public`（默认应该就是这个啦）
4 点击`create repository`创建仓库
```

##### 再建一个仓库 ovo
重复上面的步骤，把本地博客文件关联到这个仓库（按照GitHub的指示走就完全可以）：
```
git init
git status
git add .
git commit -m "start"
git remote add origin git@github.com:OooldGreen/Hugo-Sources.git
git push -u origin main
```
这里我设置了ssh代理，因为https拉取不到仓库，你懂的～
具体怎么设置代理，我看的这篇：<https://zhuanlan.zhihu.com/p/481574024>

# Jane相关
Jane Theme Preview: <https://www.xianmin.org/hugo-theme-jane/post/jane-theme-preview/#>

常用配置：<https://sixdian.com/post/jane-theme-config/>

# md相关
markdown语法：<https://daringfireball.net/projects/markdown/syntax#precode>（对不起我太菜了😭）
