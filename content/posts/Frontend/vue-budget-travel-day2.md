---
title: "Budget Travel 项目记录 Day2"
date: 2022-10-29
lastmod: 2022-10-29
draft: false
keywords: ["Vue", "项目记录", "element-ui"]
description: ""
tags: ["Vue", "项目记录"]
categories: ["Frontend", "CS"]
author: ""

# showToc: true
# TocOpen: true
# hidemeta: false
# comments: true
# description: ""
# canonicalURL: "https://canonical.url/to/page"
# disableShare: false
# disableHLJS: false
# hideSummary: false
# searchHidden: true
# ShowReadingTime: true
# ShowBreadCrumbs: true
# ShowPostNavLinks: true
# ShowWordCount: true
# ShowRssButtonInSectionTermList: true
# UseHugoToc: false
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

# 新闻页面
## 新闻分栏
用 element-ui 的标签页 tabs 做分栏，其中有属性`@tab-click="handleClick"`，点击切换的标签页 `tab.index` 返回一个数字，利用这个数字写 switch 用于请求不同分类的新闻内容，例如国内、国际、财经等。需要注意的是这个 `tab.index` 返回的看起来是一个数字，实际上是一个字符串，因此在写 switch 的时候需要给数字加上引号，我在这里报错了。
![](/image/frontend/budget-travel/news-tabs.png)
但是这个方法看起来仍然过于冗余，不知还有没有更好的方法。

## 新闻页码切换
接着页码切换就可以用前面提取过的公共组件 `MyPagination` 了，但是这里因为我把页码数据存储在了 vuex 中，导致每个不同的标签页切换页码时页码数据的变化无法分开。因此我又在公共组件 `MyPagination` 中加入了子传父的数据传输方法，就可以在每次点击相应页码，触发 `getChange` 后获取到相应标签页对应页码的数据。
```js
this.$emit('getChange', pageNum)
```
```html
<mypagination :total="total" :pageSize="pageSize" @getChange="getYaowenChange"/>
```

# 轮播图
使用了 element-ui 的走马灯效果
```html
<div class="banner">
    <el-carousel height="387px">
        <el-carousel-item v-for="item in bannerlist" :key="item.id">
        <img :src="item.img" alt="">
        </el-carousel-item>
    </el-carousel>
</div>
```
待解决问题：这里设置高度之后图片会变形，图片随着窗口自适应大小

# 关键词搜索提示
设定：当搜索框中输入关键词后，下方出现下拉框信息

方法：设置 watch 监听，出现 keyword 后，将 keyword 作为参数向后端发送请求，返回数据渲染至下拉框中
```html
<!-- 渲染下拉框 -->
<div class="show-data" v-show="isShow">
    <ul>
        <li>城市</li>
        <li v-for="(item,index) in searchList" :key="index">
        <a href="item.url">{{item.cn_name}} 
            <span class="en_name">{{item.en_name}}</span>
            <span class="hotel" v-if="item.count!==0">{{item.count}}家酒店</span>
        </a>
        </li>
    </ul>
</div>
```
```js
// 监听关键词输入
watch: {
    input(keyword) {
      if(!keyword) {
        this.isShow = false;
        this.searchList = [];
      } else {
        //发送请求，显示下拉框
    }
}
```
![](/image/frontend/budget-travel/search.png)

# 样式设计
## 盒子宽度计算
一排有固定个数的盒子，每个盒子都相同，为了让每个盒子之间有固定的间距，可以给每个子盒子设置 `margin-right: 20px`，这样第四个盒子因为也要有右边距被挤到第二行去。
![](/image/frontend/budget-travel/recommend.png)

第一种解决方法是选择 每行最右边的盒子设置右边距为 0，这种方法比较麻烦。

第二种解决方法可以给父盒子设置 `margin-right: -20px;`

```css
.container .today-recommend {
  margin-right: -20px;
  overflow: hidden;
}
```

## 渐变背景
```css
background-image: linear-gradient(90deg, rgba(40, 213, 164, .8), rgba(38, 208, 181, .8));
```

## 鼠标悬浮放大图片
```css
.img {
  overflow: hidden;
}
img {
  width: 275px;
  height: 185px;
  transition: all 1.5s;
}
img:hover {
  transform: scale(1.1);
}
```

## 多出的字用省略号替代
强制一行显示
```css
text-overflow: ellipsis;
white-space: nowrap;
overflow: hidden;
```
强制两行显示，需加入注释 `/*! autoprefixer: off */` 使 `-webkit-box-orient: vertical` 起作用
```css
overflow:hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 2;
/*! autoprefixer: off */
-webkit-box-orient: vertical;
```

# 遇到的问题
## 公共组件重复使用的数据问题
因为把pageNum存进了vuex中，后面出大问题，一个组件里引用多次分页组件的时候页码就没法写了，会互相影响，因为只有一个值。解决方法上面已经写过了，就不赘述。

## 图片请求 403
在 html 中加入
```html
<meta name='referer' content='no-referer'>
```