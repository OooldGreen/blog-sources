---
title: "Vue2 Elm 项目记录 Day1"
date: 2022-10-19T09:27:45+08:00
lastmod: 2022-11-18
draft: false 
keywords: ["vue", "elm", "项目记录"]
description: ""
tags: ["vue2", "项目记录"]
categories: ["vue", "项目记录", "CS"]
author: ""
---

一直想练手但是拖了好久的项目。

<!--more-->
此文档为vue2项目elm的练手记录，从前端页面开始做起。

# 配置
## mongodb配置

### 开启mongodb

1. `mongodb`的`bin`用终端打开

2. ` ./mongod --dbpath /usr/local/MongoDB/data/db --logpath /usr/local/MongoDB/log/mongo.log`

> Tip：如果发生错误是因为上一次没有正确的关闭mongodb，需要把.lock文件删除，再重启

### 连接navicat

![连接`navicat`](/image/frontend/vue-elm/navicat.png)

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

## 引入 SCSS
安装依赖包，出错就在后面添加 `--legacy-peer-deps`
```node
npm i node-sass sass-loader style-loader -D
```
vue-cli 会自动配置好，正常创建.scss文件就可以了。

## 引入使用 SVG 雪碧图
1. 安装依赖包 svgo 和 svg-sprite-loader
![](/image/frontend/vue-elm/dependency-svg.png)
2. 创建 svg.vue
```html
<svg>
  <defs>
    <symbol viewBox="0 0 1024 1024" id="arrow-right">
      <path d="M716.298 417.341l-.01.01L307.702 7.23l-94.295 94.649 408.591 410.117-408.591 410.137 94.295 94.639 502.891-504.76z"></path>
    </symbol>
  </defs>
</svg>
```

3. 全局引入
```vue
<!-- App.vue -->
<template>
  <svg-icon></svg-icon>
</template>
<script>
import svgIcon from './components/common/svg.vue';
export default {
  components: { svgIcon }
}
</script>
```
4. 使用
```html
<svg class="arrow_right">
  <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#arrow-right"></use>
</svg>
```

# 插槽 slot
在父组件中写入想要插在子组件中的内容，在子组件中用 slot 占好位置，编译的时候就会将父组件中写好的内容根据 slot 的位置注入到子组件中。  
当需要插入多个 slot 时，可以用 name 来指定插入的是哪一个 slot。  
```html
<!-- Home.vue -->
<HeaderTop>
  <span class="logo" slot="logo">ele.me</span> 
</HeaderTop>

<!-- head.vue -->
<header>
  <slot name="logo"></slot>
</header>
```
利用 slot 就可以在 head 组件里灵活的根据不同需要组成不同的 header 了。

# 使用 Fetch 发送请求
[参考 MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)

都是跨网络获取资源的方法，以前只学过 ajax，项目里用过 axios，今天来了解一下 fetch。

fetch 也是返回一个 Promise，但是被拒绝以后不会被标记为 reject，而是将 resolve 的返回值 ok 标记为 false，只有网络故障和请求被阻止这两种情况会标记 reject。

（1）fetch 的简单使用：
```js
fetch('https://xxx.com', {
  // 如果跨域发送 cookie
  credentials: 'include'
}).then(
  // 进行数据操作
  response => response.json()
).then(data => console.log(data))
```
（2）带有第二个参数，传递 data
```js
async function postData(url = '', data = {}) {
  credentials: 'include', // include/same-origin/omit
  mode: 'cors', // cors/no-cors/same-origin
  cache: 'default' // default/no-cache/reload/force-cache/only-if-cached
  // 配置 headers
  headers: {
    'Content-Type': 'application/json'
    // 'Content-Type': 'application/x-www-form-urlencoded',
  },
  body: JSON.stringify(data) // 数据类型和 content-type 相匹配
  // 解析 JSON 字符串
  return response.json()
}

postData('https://example.com/answer', { answer: 42 }).then(data => {
  console.log(data)
})
```

在本项目中发送请求需要带有 data。创建两个文件，一个对不同请求类型进行一些处理，另一个负责发送 fetch 请求。获得当前城市定位需要的是 GET 请求，因此先处理 GET 请求内容 —— 网址拼接字符串。
```js
// fetch.js
// 配置 fetch 

// 设置baseUrl，data，默认类型和方法
export default async (url = 'https://elm.cangdu.org', data = {}, type = 'GET', method = 'fetch') => {
  // 转换所有的请求为大写
  type = type.toUpperCase()

  // 判断请求类型
  // 如果请求类型为 GET
  if(type == 'GET') {
    let dataStr = '' //拼接字符串
    Object.keys(data).forEach(key => {
      dataStr = dataStr + key + '=' + data[key] + '&'
    })

    if(dataStr !== '') {
      // 切除最后多余的 & 符号
      dataStr = dataStr.substring(0, dataStr.lastIndexOf('&'))
      url = url + '?' + dataStr
    }
  }

  if (window.fetch && method === 'fetch') {
    let requestConfig = {
      credentials: 'include',
      method: type,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      mode: 'cors',
      cache: 'force-cache'
    }

    if (type === 'POST') {
      Object.defineProperty(requestConfig, 'body', {
        value: JSON.stringify(data)
      })
    }

    // 捕获错误
    try {
      const response = await fetch(url, requestConfig)
      const responseJson = await response.json()
      return responseJson
    } catch (error) {
      throw new Error(error)
    }
  } else {
    return new Promise((resolve, reject) => {
      let requestObj
      if (window.XMLHttpRequest) {
        requestObj = new XMLHttpRequest()
      } else {
        requestObj = new ActiveXObject
      }

      let sendData = ''
      if(type === 'POST') {
        sendData = JSON.stringify(data)
      }

      requestObj.open(type, url, true)
      requestObj.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded')

      requestObj.onreadystatechange = () => {
        if (requestObj.readyState === 4) {
          if (requestObj.status === 200) {
            let obj = requestObj.response
            if (typeof obj !== 'object') {
              obj = JSON.parse(obj)
            }
            resolve(obj)
          }
        } else {
          reject(requestObj)
        }
      }
    })
  }
}
```
```js
// getData.js
// 进行 fetch 请求

// 请求定位地址
export const cityGuess = () => fetch('/v1/cities', {
  type: 'guess'
})

// 组件中使用
cityGuess().then(data => {console.log(data)})
```

# v-for 遍历
## 遍历数组
```html
<ul>
  <li v-for="(item, index) in list" :key="index">{{item}}</li>
</ul>
```

## 还可以遍历对象
```html
<ul>
  <li v-for="(value, key, index) in list" :key="index">{{key}} --- {{value}}</li>
</ul>

<script>
  data() {
    return {
      list: {
        // key: value
        A: xx,
        B: [xx, xx, ...]
      }
    }
  }
</script>
```

# 城市按字母排序
根绝对象 key 值进行排序，这种情况可以使用计算属性。
```vue
<script>
  export default {
    computed: {
      sortGroupCity() {
        const sortObj = {}
        Object.keys(this.groupCity).sort().map(key => {
          sortObj[key] = this.groupCity[key];
        })
        return sortObj
      }
    }
  }
</script>
```

# 目标功能

- [x] 定位功能