---
title: "Vue2 Elm 项目记录 Day2"
date: 2022-11-19
lastmod: 2022-11-20
draft: false
keywords: ["vue", "elm", "项目记录"]
description: ""
tags: ["vue2", "项目记录"]
categories: ["vue", "项目记录"]
author: ""
---
# 路由

[参考](https://router.vuejs.org/zh/guide/)

## 路由跳转

### 标签跳转

使用 `<router-link>` 进行导航，通过传递 `to` 来指定链接，`<router-view>` 作为路由出口，是路由匹配的组件渲染的地方。

```html
<router-link to="/">Go to Home</router-link>
<router-link to="/city"><router-link>

<!-- 需要传参的情况 -->
<router-link :to="'/city/' + this.id"></router-link>
```

### 事件跳转（编程式路由导航）
使用 `this.$router.push('')` 或 `router.push('')`
```js
// 固定页面
this.$router.push('/home');
```
#### 带参数的情况
- 第一种方法：解构赋值
  ```js
  this.$router.push(`/city/${id}`)
  
  router.push(`/users/${username}`)
  router.push({ path: `/user/${username}` })
  ```

- 第二种方法：query 传参，相当于 get 请求，`router.push` 用法相同
  ```js
  this.$router.push({
    path: '/city',
    query: {
      id: this.id
    }
  })
  ```

- 第三种方法：params 传参，相当于 post 请求
  ```js
  this.$router.push({
    path: '/city',
    params: {
      id: this.id
    }
  })

  // router.push 中含有 path 就会忽略 params
  router.push({
    name: '/city',
    params: {
      id: this.id
    }
  })
  ```
在跳转的组件中获取参数可以使用 `this.$route.params` 或 `this.$route.query`

### 路由重定向
```js
const routes = [{ path: '/home', redirect: '/' }]
```
如果链接到一个命名路由，直接在 redirect 中指定路由名字
```js
const routes = [{ path: 'home', redirect: { name: homepage } }]
```

### 横跨历史
向前移动一条记录：`router.go(1)`  
向前移动三条记录：`router.go(3)`  
向后移动一条记录：`router.go(-1)`

# [Vuex](https://vuex.vuejs.org/zh/guide/state.html)
## State 状态
在 state 中记录 vuex 所有状态。Vuex 从根组件注入所有子组件，子组件可以通过 `this.$store` 访问。或者使用 `store.state` 获取数据。

## Mutation
### 提交 Mutation 更改 State
```js
mutations: {
  increment(state, payload) {
    state.count += payload
  }
}
store.commit('increment', 10)
```

### 使用常量替代 Mutation 事件类型
创建一个 mutation-types.js 文件，在其中定义常量
```js
// mutation-types.js
export default {
  export const SOME_MUTATION = 'SOME_MUTATION'
}
```
在 index.js 中引入
```js
// index.js
import { SOME_MUTSTION } from './mutation.types.js'

const store = {
  state: { ... },
  mutations: {
    [SOME_MUTATION] (state) {
      // 修改 state
    }
  }
}
```

# 搜索城市页面

## 跳转到城市页面
在 index.js 中引入路由
```js
const routes = [{
  path: '/city/:cityId',
  name: 'city',
  component: City
}]
```
在 Home.vue 中根据城市 id 进行跳转
```html
<router-link :to="'/city/' + this.cityId">
  <span class="guessCity">{{guessCity}}</span>
  <svg class="arrow_right">
    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#arrow-right" style="width:20px;height:20px;"></use>
  </svg>
</router-link>
```

## 获取城市信息
根据 API 文档，需要传递 id，根据城市 id 获取城市信息
```js
// getData.js
export const currentCity = cityId => fetch('/v1/cities/' + cityId)
```

在 City 组件 mounted 中，通过 `this.$route.params.cityId` 获取到从 Home 组件传过来的参数 id，用这个 id 发送请求，获取 res 为当前 id 城市信息。
```js
// City.vue
import { currentCity } from '../../service/getData'
export default {
  data () {
    return {
        cityId: '',
        cityName: ''
    }
  },
  mounted() {
    this.cityId = this.$route.params.cityId 
    currentCity(this.cityId).then(res => {
        this.cityName = res.name
    })
  }
}
```
将城市名称渲染在页面的 header 上
```html
<!-- City.vue -->
<h4 class="city-title ellipse">{{cityName}}</h4>
```

## 搜索地址
### 获取用户输入的 keyword
添加 v-model 即可
```vue
<template>
  <input type="text" placeholder="请输入学校、商务楼、地址" required v-model="keyword">
</template>
<script>
  export default {
    data() {
      return {
        keyword: ''
      }
    }
  }
<script>
```

### 根据 keyword 发送请求
在 getData.js 中
```js
// getData.js
export const getPois = (cityId, keyword) => fetch('/v1/pois', {
  city_id: cityId,
  keyword: keyword
})
```

在 City 组件中根据 keyword 发送请求，并将结果保存在 data 中，用 li 渲染在页面中，如果没有搜索结果显示提示。
```vue
<template>
<section class="search">
  <ul class="searchUl">
    <li v-for="(item, index) in poisMsg" :key="index">
      <h4 class="search-title">{{item.name}}</h4>
      <p class="address ellipse">{{item.address}}</p>
    </li>
  </ul>
  <footer class="no-result" v-if="noresult">很抱歉！无搜索结果</footer>
</section>
</template>

<script>
import { currentCity, getPois } from '../../service/getData'

export default {
  data () {
    return {
        // 搜索历史在提交关键词后隐藏
        historyHide: false,
        // 存储搜索出的结果
        poisMsg: [],
        // 无搜索结果
        noresult: false
    }
  },
  methods: {
    searchPois() {
        // 隐藏搜索历史
        this.historyHide = true
        getPois(this.cityId, this.keyword).then(res => {
            if(res.length === 0) {
                // 如果结果为空，没有搜索结果时显示提示
                this.noresult = true
            } else {
                this.noresult = false
                this.poisMsg = res
            }
        })
    }
  }
}
</script>
```

### 搜索历史
搜索历史可以存储在 localStorage 中。进入页面时从 localStorage 中获取历史搜索记录
```js
export default {
  data() {
    history: []
  },
  methods: {
    // 得到历史搜索记录
    initData() {
      if(getStore('history')) {
        this.history = JSON.parse(getStore('history'))
      } else {
        this.history = []
      }
    }
  },
  mouted() {
    // 初始化搜索记录
    this.initData()
  }
}
```

跳转到下一个页面之前将用户点击的地址记录在 localStorage 中
```html
<!-- 给搜索结果绑定点击事件 -->
<li v-for="(item, index) in poisMsg" :key="index" @click="nextPage(item)"></li>
```
```js
export default {
  data() {
    return {
      // 记录历史
      history: []
    }
  },
  method: {
    nextPage (place) {
      // 如果存在历史记录，搜索是否存在该记录
      if (this.history) {
        let checkRepeat = false
        this.history.forEach((item) => {
          if (place.geohash === item.geohash) {
            checkRepeat = true
          }
        })
        if (!checkRepeat) {
          this.history.push(place)
        }
      } else {
        this.history.push(place)
      }
      // 存储到 localStorage
      setStore('history', this.history)
    }
  }
}
```

## 一些小优化
### 封装 localStorage 的存储等方法
添加一个优化 js
```js
// mUtils.js
// 获取 localStorage
export const getStore = name => {
  if (!name) return;
  return window.localStorage.getItem(name)
}

// 存储 localStorage
export const setStore = (name, content) => {
  if (!name) return;
  window.localStorage.setItem(name, content)
}

// 移除 localStorage
export const removeStore = name => {
  if (!name) return;
  window.localStorage.removeItem(name)
}
```
在需要的时候直接引入
```js
import { getStore, setState } from '../../config/mUtils'
this.history = getStore('history')
setStore('history', this.history)
```

### 隐藏历史记录
当用户提交文本框内容，隐藏历史记录显示搜索记录；当输入框失去焦点时，如果用户没有输入则重新显示历史记录。可以给 input 框添加 `@blur` 事件
```vue
<template>
  <input type="search"
    placeholder="请输入学校、商务楼、地址"
    required
    v-model="keyword"
    @blur="hideHistory"
  >
</template>

<script>
  methods: {
    hideHistory () {
      if (this.keyword.trim()) {
        this.historyHide = true
      } else {
        this.historyHide = false
        // 清空搜索结果
        this.poisMsg = []
      }
    }
  }
</script>

```

### 清空历史记录
```html
<!-- City.vue -->
<footer class="history-clear" v-if="this.history.length!==0" @click="clearHistory">清空所有</footer>
```
```js
clearHistory () {
  // 移除 localStorage 存储
  removeStore('history')
  // 初始化数据
  this.initData()
}
```

```js
// mUtil.js
export const removeStore = name => {
  if (!name) return;
  window.localStorage.removeItem(name)
}
```

# 目标功能

- [x] 定位功能
- [x] 选择城市
- [x] 搜索地址

<!-- - [ ] 展示所选地址附近商家列表
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
- [ ] 上传头像 -->