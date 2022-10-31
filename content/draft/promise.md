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

从是什么，怎么用开始讲起啦，然后讲讲具体有哪些方法和应用场景，最后再浅提一下`async/await`

<!--more-->

# 为什么要有Promise对象

一句话：解决回调地狱(callback hell)

如果你想进行一个异步操作，比方说，有先后顺序的打印两个结果，你会怎么写？

也许你可以使用回调函数（callback），因为回调函数必须依赖另一个函数执行，我们可以借此确保打印的先后顺序。定时器就是一个常见的回调函数。
```js
// 1s后输出callback
setTimeout(function () { 
    console.log('callback')
}, 1000);
```


# Promise对象是什么
Promise是一个异步操作，通俗来说就是Promise里面的程序运行完根据返回值再进行下一步操作，这样不用一直在函数体里一直嵌套回调。

一共有三种状态：`PENDING`（待定）、`FULFILLED/RESOLVED`（成功）、`REJECTED`（失败）。这是一个“承诺”，从一开始的`PENDING`开始，状态一旦改变便不会再变，成败只在一线之间（bushi。所以最后结果就是返回`RESOLVED`或者`REJECTED`，一般来说返回`RESOLVED`进行下一步，返回则`REJECTED`抛出错误。

# Promise对象怎么用

## 基本用法
用`new`实例化，接住`resolve`或`reject`
```js
const p1 = new Promise((resolve, reject) => {

})
```

# Promise有哪些方法捏

## then


## catch

## all

## track

# async/await又是啥？


## 参考
- [JavaScript高级程序设计（第4版）](https://book.douban.com/subject/35175321/)
- [阮一峰老师的ES6入门](https://es6.ruanyifeng.com/#docs/promise)




