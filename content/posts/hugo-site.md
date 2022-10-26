---
title: "关于配置博客"
date: 2022-10-17T10:11:51+08:00
lastmod: 2022-10-18T10:11:51+08:00
draft: false
keywords: ["Hugo", "jane"]
description: ""
tags: ["Jane", "blog", "域名", "markdown"]
categories: ["Blog"]
author: ""

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: true

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---

从小到大都对属于自己的东西格外着迷，可能是骨子里的控制欲作祟吧，于是突发奇想地想要拥有属于自己的博客。

就当是我在这动荡时代的自留地吧，一间抵御风寒酷暑的小木屋。

这篇博客一共分为四个部分：用`hugo`搭建个人博客，用`vercel`部署博客，自定义域名，最后是一些有用的文档内容。

<!--more-->

# Hugo搭建

## 下载Hugo
参考[官方文档](https://gohugo.io/getting-started/installing/)
### 方法一：二进制（MACOS）


1. 进入[Hugo Release](https://github.com/gohugoio/hugo/releases)
2. 下载最新版本，解压缩（记得要下载`extend`版本）
3. 把`hugo`文件放进`/usr/local/bin`
4. 输入`hugo version`检查版本号，出现版本号，安装成功


### 方法二：Homebrew
或者更简单，如果你有`homebrew`，直接：
```text
brew install hugo
```

## 建立新网站

```text
hugo new site quickstart
```
这句指令会生成一个名为`quickstart`的文件夹，如下图：
{{< figure src="/image/quickstart.png" title="quickstart文件夹">}}

## 选择喜欢的主题
按照hugo的提示选择[主题](https://themes.gohugo.io/)，我选择的主题是[jane](https://github.com/xianmin/hugo-theme-jane)，具体步骤：
1. 直接下载压缩包解压
2. 经过一个改名字的动作（需要把文件名称从`hugo-theme-jane`改为`jane`）
3. 再把文件夹放在`quitckstart > themes`文件夹下，就阔以啦
4. 接着我按照jane的说明文档copy了一些网站样例和默认设置：
    ```text
    cp -r themes/jane/exampleSite/content ./
    cp themes/jane/exampleSite/config.toml ./
    ```

当然，也可以创建自己的主题：
```text
hugo new theme <THEMENAME>
```

## 生成我的第一支博客
在我的博客文件夹下输入以下代码：
```bash
hugo new post/my-first-post.md
```

> Tips:
>
>  jane配置的博客目录在`post`，不同主题可能有细微差别
> 
>  名字一定要加后缀！起名字最好就是英文小写+短横线～

# 部署

## 部署到Github

### 先有一个Github账号


1. 进入[github](https://github.com/)（此处可能需要🪜）
2. 输入邮箱，点击`Sign up for GitHub`创建账号
3. 然后登录就好啦


### 创建新的repository

1. 点击右上角加号`+` -> `new repository`
2. 输入你的`repository name`，例如`myblog`
3. 记得检查一下是否选择`public`（默认应该就是这个啦）
4. 点击`create repository`创建仓库

### 把本地博客文件关联到这个仓库
这步按照`GitHub`创建仓库后的指示走就完全可以
```text
git init
git status
git add .
git commit -m "start"
git remote add origin git@github.com:OooldGreen/Hugo-Sources.git
git push -u origin main
```
这里我设置了`ssh`代理，因为`https`拉取不到仓库，你懂的～
具体怎么设置代理，我看的这篇：<https://zhuanlan.zhihu.com/p/481574024>

# 通过Vercel生成网站
[Vercel](https://vercel.com/login?next=)在这里，它方便快捷。
当然如果用`github pages`或者别的也可以～

1. 先用`GitHub`登录，进入`DashBoard`
2. 点击`Create a New Project`
3. 关联`GitHub`，导入`myblog`仓库，点击`Deploy`
4. 出现`Congratulations！`就成功啦，进入`DashBoard`就可以点击进入自己的网站啦

# 自定义域名
如果你觉得自动生成的域名不够炫酷，还可以选择自定义域名。只要你有心，不花钱钱也是可以做到的哦（0v0）。我全程参考了[necolas小站](https://blog.nekolas.cafe/posts/hugo/hugo-custom-domain/)的教程😁

如果想完成此步，需要准备两个网站：

[1. 免费租域名](https://my.freenom.com)

[2. 管理域名的腾讯云](https://console.dnspod.cn/dns)

## 选择你喜欢的域名
1. 在搜索框中搜索域名是否可用
2. 复制全部域名再搜索一遍
3. 此时购物车`+1`，点击`checkout`
4. 拥有时长选择`12months`，结账完事～

页面记得留着，等会儿还得用

## 在腾讯云配置
1. 腾讯云需要实名验证，验证完之后进入“我的域名”
2. 输入域名，进入`DNS`检测，检测会出问题，复制正确的配置
3. 回到`freenom`粘贴配置
4. 再回到腾讯云重新检测，这时就没有问题啦

## 在Vecel中配置
1. 点击`Settings -> Domains `
2. 输入你已经拥有的域名，点击`add`，默认选项即可
3. 此时会显示域名不可用
4. 在腾讯云中`点击域名 -> 添加记录`
5. 按照提示`vecel`中的`"type"`, `"name"`, `"value"`进行配置，分别对应“记录类型”, “主机记录“, ”记录值“

最后耐心等待一会儿就可以看到自己专属的域名啦～


# Jane相关
Jane Theme Preview: <https://www.xianmin.org/hugo-theme-jane/post/jane-theme-preview/#>

常用配置：<https://sixdian.com/post/jane-theme-config/>

# md相关
markdown语法：<https://daringfireball.net/projects/markdown/syntax#precode>（对不起我太菜了😭）
