---
title: "Budget Travel 项目记录 Day2"
date: 2022-10-29
lastmod: 2022-10-29
draft: false
keywords: ["vue", "项目记录", "element-ui"]
description: ""
tags: ["vue", "项目记录"]
categories: ["vue","项目记录"]
author: ""

showToc: true
TocOpen: true
hidemeta: false
comments: true
description: ""
# canonicalURL: "https://canonical.url/to/page"
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

穷游小项目跟练第二天。发现这虽然只是个随便练手的小项目，但是真的能练到很多理论上看过但没用过的功能（对我这种没有实战经验的小菜鸡来说）。

<!--more-->
# 制作n宫格图片

这是插在首页 Home 的轮播图下面的。

## 创建组件并布局

这步主要是用 list 做出一个 n 宫格，这个 n 由后台请求回来的图片数决定。假设一排 5 张图片，所以图片都是 5 的倍数（练习里就是有 20 张图片）。

因此，写样式时用 flex 布局，先准备一个容器，规定好宽度，其中每一个 li 宽度为 20%，并用绝对定位定小标题的位置。
```css
.container {
    width: 1200px;
    margin: 0 auto;
    margin-top: 30px;
}
.list {
    display: flex;
    flex-wrap: wrap;
}
.list li {
    position: relative;
    box-sizing: border-box;
    width: 20%;
}
.list span {
    position: absolute;
    right: 20px;
    bottom: 0px;
}
```
## 获取数据并渲染

在首页组件 Home 获取图片数据，created 钩子里获取第一页图片，还要写一个根据页码变化获取图片。

在 data 中准备好请求回来的数据的容器 `list: []`，以及总页数 total

在 methods 中写请求的公共方法：
```js
// Home.vue
getBlueberry(pageNum) {
    this.$api.getBlueberry({
        blueBerryjam_id: pageNum
    }).then((res) => {
        // console.log(res.data)
        if (res.status === 200) {
            this.list = res.data.blueBerryJam;
            this.total = res.data.maxPage;
        } else {
            return this.$message.error('获取数据失败:(')
        }
    })
}
```

created 钩子里就直接写请求第一页
```js
// Home.vue 
created: {
  this.getBlueberry(1)
}
```

通过 props 父传子把请求回来的 list 和 总页数 total 传递给 n 宫格子组件：
```html
<!-- Home.vue -->
<Blueberry :list="list" :total="total"/>
```

在 n 宫格子组件中，用 v-for 遍历图片创建 li 列表
```html
<!-- Blueberry.vue -->
<div class="container"> 
  <ul class="list">
    <li v-for="(item, index) in list" :key="index">
        <div class="item">
            <img :src="item.img" :alt="item.title">
            <span>{{item.title}}</span>
        </div>
  </li>
</ul>
  </div>
```

## 根据页码变化获取图片
页码存储在 vuex 中，在 watch 中监听页码的变化：
```js
'$store.state.pageNum'(newPageNum) {
    // console.log(newPageNum)
    // 页码改变重新请求蓝莓酱页码
    this.getBlueberry(newPageNum)
}
```

## 换页回到顶部
在 watch 中监听页码的变化，一旦变化就用 scrollTop
```js
document.documentElement.scrollTop = 500 + 'px';
```

# 二次封装分页组件

## 创建组件
获取 element-ui 本身的分页
```html
<el-pagination
    background
    layout="prev, pager, next"
    :total="1000"
>
</el-pagination>
```
在 `/components/` 文件夹中建立自己的分页组件
```html
<el-pagination
    background
    layout="total, prev, pager, next, jumper"
    :total="total"
    :pageSize="pageSize"
    @current-change="pageChange"
>
</el-pagination>
```

## 配置参数和方法
在 props 中设置 total 总页数 和 pageSize 每页数据个数都可以改变，但是都要设定类型为数字，且需要设置默认值，这样即使没有数据也不会不显示
```
props: {
    total: {
        type: Number,
        default: 100
    },
    pageSize: {
        type: Number,
        default: 10
    }
}
```
在 methods 中定义监听页码改变的方法
```js
methods: {
    pageChange(pageNum) {
        // 只传输页码数据，不请求数据
        this.$store.commit('getPageNum', pageNum)
    }
}
```
我直接把页码数据 pageNum 存储在了 Vuex 中，这样需要在需要用到页码的组件中监听 vuex 中页码的变化（上面写过的代码就不再写一遍了）。其实只要能传递给需要用的组件就可以了，子传父可以用 $emit 。
```js
// store/index.js
const store = new Vuex.Store({
  state: {
    pageNum: 0
  },
  mutations: {
    getPageNum(state, payload) {
      state.pageNum = payload
    }
  }
})
```

## 使用
封装好之后，如果要使用，引入后直接传入需要的 pageSize 和 total 这两个变量即可
```html
<Pagination :total="total" :pageSize="pageSize"/>
```
