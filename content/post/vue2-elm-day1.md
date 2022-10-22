---
title: "Vue2 Elm 项目记录Day1"
date: 2022-10-19T09:27:45+08:00
lastmod: 2022-10-19T09:27:45+08:00
draft: true
keywords: ["vue", "elm", "项目记录"]
description: ""
tags: ["vue2", "项目记录"]
categories: ["vue", "项目记录"]
author: ""

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: true
reward: false
mathjax: true

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---

一直想练手但是拖了好久的项目。

<!--more-->
此文档为vue2项目elm的练手记录，从前端页面开始做起，时间：2022.9.7 晚上9:30

## mongodb配置

### 开启mongodb

1. `mongodb`的`bin`用终端打开

2. ` ./mongod --dbpath /usr/local/MongoDB/data/db --logpath /usr/local/MongoDB/log/mongo.log`

> Tip：如果发生错误是因为上一次没有正确的关闭mongodb，需要把.lock文件删除，再重启

### 连接navicat

![连接`navicat`](/image/vue-elm/navicat.png)

若果第一步开启好了，点击左下角的test很快就会提示测试连接成功，直接确定就好了

### 导入数据库

1. 对于`.metadata.json`文件：  `mongoimport -d elm_db -c activities --file activities.metadata.json`

2. 对于`.bson`文件： ` mongorestore -d elm_db -c activities activities.bson`

3. 在`navicat`中刷新就可以看到数据被导入了

### 正确的关闭mongodb

使用`Ctrl + C`

> `Warning`：千万不能使用`kill -9 <pid>`,因为`MongoDB`使用`mmap`方式进行数据文件管理，也就是说写操作基本是在内存中进行，写操作会被每隔60秒(`syncdelay`设定)的`flush`到磁盘里。如果在这60秒内`flush`处于停止事情我们进行`kill -9`那么从上次`flush`之后的写入数据将会全部丢失。

### 其他mongodb操作

当前数据库：`db`

展示所有数据库：`show dbs`

切换到`admin`数据库：`use admin`

## Eslint配置
- 关闭校验

在根目录下创建vue.config.js
```js
moduel.export = {
  // 关闭eslint
  lintOnSave: false
}
```

- 检验规则配置
```js
// .eslintrc.js
rules: {
  // 禁止空格报错检查
  'generator-star-spacing' : 'off',
  // 不校验组件名
  'vue/multi-word-component-names': 'off',
  // 没用过的不报错
  'no-use-before-define': 'off'
}
```

## 路由配置
创建`Home.vue`，在路由js中引入并配置：
``` js
const routes = [
  {
    path: '/home',
    name: 'home',
    component: Home,
    meta:{
      // 需要被缓存
      keepAlive: true
    }
  }
]
```

## 封装公共组件


## 定位功能
通过跨域获取当前地理位置


## 目标功能

- [ ] 定位功能
- [ ] 选择城市
- [ ] 搜索地址
- [ ] 展示所选地址附近商家列表
- [ ] 搜索美食
- [ ] 根据距离、销量、评分、特色菜、配送方式等进行排序和筛选
- [ ] 餐馆视频列表页
- [ ] 购物车功能
- [ ] 店铺评价页面
- [ ] 单个食品详情页面
- [ ] 商家详情页
- [ ] 登录、注册
- [ ] 修改密码
- [ ] 个人中心
- [ ] 发送短信、语音验证
- [ ] 下单功能
- [ ] 订单列表
- [ ] 订单详情
- [ ] 下载app
- [ ] 添加、删除、修改收货地址
- [ ] 账户信息
- [ ] 服务中心
- [ ] 红包
- [ ] 上传头像