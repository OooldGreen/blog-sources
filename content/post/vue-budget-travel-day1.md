---
title: "Budget Travel项目记录 Day1"
date: 2022-10-19T20:54:07+08:00
lastmod: 2022-10-19T20:54:07+08:00
draft: false
keywords: ["vue", "项目记录", "element-ui"]
description: ""
tags: ["vue", "项目记录"]
categories: ["vue","项目记录"]
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
mathjax: false

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---
穷游小项目跟练第一天。
<!--more-->

## 创建vue项目

1. 控制台方法  
    创建项目：
    ```text
    vue create filename
    ```
    启动项目：
    ```text
    cd filename
    npm run server
    ```
2. vue ui方法  
    打开vue-ui，直接进行创建和配置

## 配置路由拦截

配置路由前置守卫用于校验用户是否有登录权限。
先在路由界面配置`meta`判断是否需要权限
```js
meta: { isLogin: true }
```
再配置路由守卫进行判断，如果需要登录权限，则判断是否已经登陆，若没有登录跳转登录页面，否则放行。
```js
// 路由拦截：路由前置守卫
router.beforeEach((to, from, next) => {
  // 1.判断当前的路由是否需要登录
  if (to.meta.isLogin) {
    // 2.判断当前的用户状态是否已登录
    const token = ''
    if (token) {
      next()
    } else {
      next('/login')
    }
  } else { // 不需要登录
    next()
  }
})
```

## 写登录页面
### 引入element-ui
#### 全局引入
在`main.js`中写入以下代码
```js
import Vue from 'vue';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import App from './App.vue';

Vue.use(ElementUI);

new Vue({
  el: '#app',
  render: h => h(App)
});
```

#### 按需导入
全局引入并不是最好的方法，我们可以使用按需引入来减小项目体积，优化项目。  

首先借助`babel-plugin-component`插件，在`babel.config.js`中写入以下代码：
```js
module.exports = {
  "presets": [
    "@vue/app"
  ],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      }
    ]
  ]
}
```
接着开始部分引入。可以单独写一个`element.js`来做引入，引入单独的组件，例如登录界面所需的表单组件和按钮。
```js
// element.js
import Vue from 'vue'
import {
  Button,
  Form,
  FormItem,
  Input,
} from 'element-ui'

Vue.use(Button)
Vue.use(Form)
Vue.use(FormItem)
Vue.use(Input)
```
最后在`main.js`中引入`element.js`文件。
```js
// main.js
import './plugins/element'
```

### 创建一个简易的登录接口
第一步：安装`express`和`jwt`
```text
npm i express -S
```

第二步：在根目录下创建服务器文件夹`server`

第三步：在`server`中创建`index.js`
- 引入`express`和`cors`
    ```js
    // 引入express和cors
    const express = require('express')
    const app = express()

    const cors = require('cors')
    app.use(cors())
    ```
- 开启服务端口
    ```js
    app.listener(8888, () => {
        console.log('server running at http://127.0.0.1:8888')
    })
    ```
- 创建登录接口
    ```js
    app.get('/login', (req, res) => {
    let user = req.query.username;
    let pwd = req.query.pwd;
    res.send({
        info: 'success',
        status: 200,
        data: {
            user,
            pwd
        }
    })
    ```
第四步：打开`server`文件夹的终端窗口，`nodemon`启动（安装`nodemon`可以实时更新不用一直关闭重启）

### 生成token
第一步：安装`jwt`
```js
npm i jsonwebtoken -S
```
第二步：自定义密钥？并引入
```js
// secretKey.js
module.exports = {
    secretKey: '20223jiayou'
}

// index.js 引入密钥
const secret = require('./secretkey')
```

第三步：生成token  

语法：`jwt.sign('数据','密钥','过期时间')`  
过期时间: `{ expiresIn: '1day/1h/10' }` 默认单位为秒
```js
// 生成token标识
const jwt = require('jsonwebtoken')

const token = jwt.sign({ user, id: xxx }, secret.secretKey, { expiresIn: 20 })
res.send(token)
```

### 存储用户数据（vuex和localstorage）


## 遇到的问题

### beforeEach is not a function
使用导航守卫时页面报错了：`beforeEach is not a function`

检查后发现我直接daochule`export default new Router({...})`，然后写的`beforeEach`

应该先new一个实例，给声明的路由实例添加方法才对：`const router = new Router({ routes })`

### vuex版本问题
用的`vue2`但是`vuex`装的是4，报错，检查`this.$store`输出为`undefined`

改成`vue2`对应的`vuex3`： `npm install vuex@3`

然后babel报错：`npm install @babel/core @babel/preset-env`
