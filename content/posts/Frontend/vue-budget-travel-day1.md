---
title: "Budget Travel 项目记录 Day1"
date: 2022-10-19T20:54:07+08:00
lastmod: 2022-10-19T20:54:07+08:00
draft: false
keywords: ["vue", "项目记录", "element-ui"]
description: ""
tags: ["vue", "项目记录"]
categories: ["vue","项目记录"]
author: ""

showToc: true
TocOpen: true
draft: false
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
穷游小项目跟练第一天。
<!--more-->

# 创建vue项目

1. 控制台方法  
    创建项目：
    ```
    vue create filename
    ```
    启动项目：
    ```
    cd filename
    npm run server
    ```
2. vue ui方法  
    打开vue-ui，直接进行创建和配置

# 配置路由拦截

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
# 写根目录页面

## 引入element-ui
### 全局引入
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

### 按需导入
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
接着开始部分引入。可以单独写一个`element.js`来做引入，引入单独的组件，例如home页面写了一个菜单栏和登录按钮，引入如下。以后需要什么组件也都要引入。
```js
// element.js
import Vue from 'vue'
import {
  Button,
  Menu,
  MenuItem
} from 'element-ui'

Vue.use(Button)
Vue.use(Menu)
Vue.use(MenuItem)
```
最后在`main.js`中引入`element.js`文件。
```js
// main.js
import './plugins/element'
```

## 菜单栏
把`myMenu`作为布局`layout.vue`的一个组件，在`myMenu.vue`中用`el-menu`做菜单栏。

- 菜单栏页面内容
  ```vue
  <!-- myMenu.vue -->
  <template>
  <!-- 导航栏 -->
    <div class="nav">
      <el-menu
        :default-active="$route.path"
        class="el-menu-demo"
        mode="horizontal"
        @select="selectMenu"
      >
        <el-menu-item index="/">首页</el-menu-item>
        <el-menu-item index="/news">新闻</el-menu-item>
        <el-menu-item index="/about">我的</el-menu-item>
        <el-menu-item index="/travel">旅游</el-menu-item>
    </div>
  </template>
  ```

- 在布局组件中引入
  ```vue
  <!-- layout.vue -->
  <template>
    <div>
      <myMenu/>
    </div>
  </template>

  <script>
  import myMenu from '../components/MyMenu.vue'
  export default {
    components: { myMenu }
  }
  </script>
  ```

- 登录按钮（需要先写好登录页面）
  描述：点击页面右上角登录按钮跳转登录页面，登录成功跳回首页，右上角按钮显示退出登录。
  ```vue
  <template>
    <div>
      <!-- 导航右侧内容 -->
        <div class="nav-right">
          <!-- 没登录显示登录按钮，已登录显示退出登录template -->
          <el-button v-if="!userInfo.username" size="small" @click="login">登录</el-button>
        </div>
      </el-menu>
    </div>
  </template>


  <script>
  export default {
    methods: {
      login() {
          this.$router.push('/login')
      }
    }
  };
  </script>
  ```

- 登出按钮
  描述：点击退出登录清空vuex中和存储在本地的用户信息，跳回首页。
  
  ```vue
  <template>
    <div>
      <!-- 导航右侧内容 -->
        <div class="nav-right">
          <!-- 没登录显示登录按钮，已登录显示退出登录template -->
          <template v-else>
              <span>欢迎你，"{{userInfo.username}}"</span>
              <el-button size="small" @click="logout">退出登录</el-button>
          </template>
        </div>
      </el-menu>
    </div>
  </template>

  <script>
  import { mapState, mapMutations } from 'vuex'
  export default {
    methods: {
      ...mapMutations('loginModule', ['clearUser']),
      logout() {
          // 清空vuex和本地用户数据
          this.clearUser(),
          localStorage.removeItem('userInfo')
          // 跳转路由
          this.$router.push('/')
      }
    },
    computed: {
      ...mapState('loginModule', ['userInfo'])
    }
  };
  </script>
  ```

# 写登录页面

## 创建一个简易的登录接口
第一步：安装`express`和`jwt`
```
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

## 生成token
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

## 存储用户数据（vuex和localStorage）
### vuex存储
在`store`中设置`token`和清空用户信息
```js
state: {
    userInfo: {
        username: '',
        token: null
    }
},
mutations: {
    // 设置token
    setUser(state, payload) {
        state.userInfo = payload;
    },
    // 清空用户信息
    clearUser(state) {
        state.userInfo = {
            username: '',
            token: null
        }
    }
}
```
在`Login.vue`配置存储一个对象包含`token`和用户名，存储在`vuex`中
```js
//Login.vue
// 存储数据并用vuex共享资源
let obj = {
    token: res.data.token,
    username: jwt(res.data.token).user
}
this.$store.commit('setUser', obj)
```
### 存储到localStorage
```js
// Login.vue
localStorage.setItem('userInfo', JSON.stringify(obj)) //对象需要转化成字符串
```

# 遇到的问题
## beforeEach is not a function
使用导航守卫时页面报错了：`beforeEach is not a function`

检查后发现我直接daochule`export default new Router({...})`，然后写的`beforeEach`

应该先new一个实例，给声明的路由实例添加方法才对：`const router = new Router({ routes })`

## vuex版本问题
用的`vue2`但是`vuex`装的是4，报错，检查`this.$store`输出为`undefined`

改成`vue2`对应的`vuex3`： `npm install vuex@3`

然后babel报错：`npm install @babel/core @babel/preset-env`

## 报错`Uncaught (in promise) NavigationDuplicated`
发现是vue-router跳转到自己的时候会报错，查看我的vue-router版本已经是`3.5.1`，所以没有按网上说的装`3.0`版本，最后解决方法是在`main.js`中加入以下代码
```js
import Router from 'vue-router'
Vue.use(Router)

const originalPush = Router.prototype.push
Router.prototype.push = function push(location) {
  return originalPush.call(this, location).catch(err => err)
}
```
