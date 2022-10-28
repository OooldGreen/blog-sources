---
title: "Promise对象"
date: 2022-10-18T15:35:31+08:00
lastmod: 2022-10-18T15:35:31+08:00
draft: true
keywords: ["frontend", "promise", "js", "javascript", "es6"]
description: ""
tags: ["JS", "ES6"]
categories: ["ES6"]
author: ""

showToc: true
TocOpen: true
draft: true
hidemeta: false
comments: true
description: ""
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: false
---

汇总`Promise`对象的由来、用法、方法和`async/await`

当然是从是什么，怎么用开始讲起啦，然后讲讲具体有哪些方法和应用场景，最后再浅提一下`async/await`

<!--more-->

# 为什么要有Promise对象

一句话：解决回调地狱(`callback hell`)

如果你要在函数中嵌套函数形成异步任务，就会形成函数里里外外很多层的情况，非常的麻烦，不易读也不易于维护。

而如果有了Promise，就可以使用直接返回回调，还可以使用不同的方法使用这些回调，这样代码看起来就简洁明了多啦～

# Promise对象是什么

是一个异步编程的解决方案。一共有三种状态：`PENDING`、`FULFILLED/RESOLVED`、`REJECTED`


# Promise对象怎么用


# 还有哪些方法捏


# async/await又是啥？


## 参考
- [JavaScript高级程序设计（第4版）](https://book.douban.com/subject/35175321/)
- [阮一峰老师的ES6入门](https://es6.ruanyifeng.com/#docs/promise)




