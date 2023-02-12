---
title: "Vue2 Elm 项目记录 Day4"
date: 2022-11-28
lastmod: 2023-02-06
draft: false
keywords: ["vue", "elm", "项目记录"]
description: ""
tags: ["vue2", "项目记录"]
categories: ["vue", "项目记录"]
author: ""
---
# 从 Msite 导航进入的商家列表
## 从 url 得到点击的分类 id

```html
<router-link
    v-for="foodItem in item" 
    :key="foodItem.id" 
    :to="{ path: '/food', 
           query: { 
             geohash, 
             title: foodItem.title, 
             restaurantCateId: getCateId(foodItem.link) 
           }
        }" 
></router-link>
```
```js
// 得到餐厅分类id
getCateId(url) {
  let urlData = decodeURIComponent(url.split('=')[1].replace('&target_name',''));
  if (/restaurant_category_id/gi.test(urlData)) {
    return JSON.parse(urlData).restaurant_category_id.id
  } else {
    return ''
  }
}
```

## 下拉菜单制作

### CSS 类的动态选择
可以使用 计算属性 或者 通过模版代码孔氏是否添加CSS样式

1. 方法一：计算属性
```html
<template>
  <section :class="category_active"></section>
</template>
<script>
  export default {
    data() {
      return {
        attachRed: false
      }
    }
    computed: {
      category_active: function () {
        return { red: this.attachRed, grey: !this.attachRed }
      }
    }
  }
</script>
```

2. 方法二：模版代码
```html
<template>
  <section :class="{ category_active: restaurant_category_id == item.id }"></section>
</template>
```


### CSS 超出部分滑动显示
设置高度，并设置 overflow 即可实现超出的部分滑动

```css
.cate-right {
  height: 27rem;
  overflow: auto;
}
```

### 下拉菜单隐藏显示

点击图标旋转，小三角旋转 180 度，变成蓝色

```css
.sort-svg {
  transform: rotate(180deg);
  transition: all .3s;
  fill: $blue
}
```

使用 transition 标签实习点击显示隐藏菜单

```html
<!-- html部分 -->
<section @click="chooseType('food')">
  <transition name="showlist">
    <!-- 菜单内容 -->
    <section v-show="sortBy==='food'"></section>
  </transition>
</section>
```
```css
/* css部分 */
/* 平滑过渡效果 */
.showlist-enter-active,
.showlist-leave-active {
  transition: all .3s;
  transform: tranlateY(0);
}
.showlist-enter,
.showlist-leave-to {
  opacity: 0;
  transform: translateY(-100%);
}
```
```js
// js部分
data() {
  return {
    sortBy: '',
  }
}
methods: {
  // 判断点击菜单类型，按类型显示菜单
  chooseType(type) {
    // console.log(type)
    // 当sorBy无值，菜单显示；当sortBy有值，隐藏菜单
    if (this.sortBy !== type) {
      this.sortBy = type
    } else {
      this.sortBy = ''
    }
  }
}
```

# 补充根据属性值获取商家数据
在 Food.vue 中给 shoplist 传递分类数据
```html
<div class="shop-list">
  <ShopList :restaurantCateId="restaurantCateId"
            :geohash="geohash"
            :restaurantCateIds="restaurantCateIds"
            :sortByType="sortByType"
            :deliveryMode="deliveryMode"
            :confirmSelect="confirmSelect"
            :supportIds="support_ids"
            v-if="latitude">
  </ShopList>
</div>
```

在 ShopList.vue 中按照分类数据重新请求商家列表
```js
export default {
    props: ['restaurantCateId', 'restaurantCateIds', 'sortByType', 'deliveryMode', 'supportIds', 'confirmSelect', 'geohash'],
  methods: {
    // 属性值发生变化时重新请求商家列表
    async changeProp() {
      // 加载等待
      this.showLoading = true
      this.offset = 0
      // 重新请求数据
      let res = await shopList(this.latitude, this.longitude, this.offset, '', this.restaurantCateIds, this.sortByType, this.deliveryMode, this.supportIds)
      this.hideLoading()
      // 考虑到本地模拟数据是引用类型，所以返回一个新的数组
      this.shopList = [...res]
    }
  },
  watch: {
    // 监听几个参数的改变，一旦发生变化按照属性值重新获取数据
    restaurantCateIds: function (value) {
      this.changeProp()
    },
    // 监听排序方式的改变
    sortByType: function (value) {
      this.changeProp()
    },
    // 监听确认按钮点击
    confirmSelect: function (value) {
      this.changeProp()
    }
  }
}
```


# 筛选

# 目标功能

- [x] 定位功能
- [x] 选择城市
- [x] 搜索地址
- [x] 展示所选地址附近商家列表
- [x] 搜索美食
- [ ] 根据距离、销量、评分、特色菜、配送方式等进行排序和筛选

<!-- - [ ] 餐馆视频列表页
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