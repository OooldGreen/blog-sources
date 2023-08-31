---
title: "Vue2 Elm 项目记录 Day4"
date: 2022-11-28
lastmod: 2023-02-06
draft: false
keywords: ["vue", "elm", "项目记录"]
description: ""
tags: ["vue2", "项目记录"]
categories: ["vue", "项目记录", "CS"]
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
菜单布局分三部分，配送方式、商家属性和清空确认按钮；  
用户选择配送方式和商家属性后根据选择进行商铺筛选，用户选中的选项前面的图标会变为对勾✅，字体颜色变为蓝色；  
在确认按钮部分会出现计数； 


```html
<transition name="showlist">
  <section v-show="sortBy=='activity'" class="sort-detail-type activity-container">
    <section style="width: 100%">
      <header class="activity-header">配送方式</header>
      <ul class="activity-filter">
        <li class="filter-li" v-for="(item, index) in this.delivery" :key="index" @click="selectDeliveryMode(item.id)">
          <svg :style="{ opacity: (item.id == 0) && (delivery_mode !==0) ? '0' : '1' }">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" :xlink:href="deliveryMode == item.id ? '#selected' : '#fengniao'"></use>
          </svg> 
          <span :class="{ select_filter: deliveryMode == item.id }">{{item.text}}</span>
        </li>
      </ul>
    </section>
    <section style="width: 100%">
      <header class="activity-header">商家属性（可以多选）</header>
      <ul class="activity-filter">
        <li class="filter-li" v-for="(item, index) in this.activity" :key="index" @click="selectSupportIds(index, item.id)">
          <svg v-show="supportIds[index].status">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#selected"></use>
          </svg>
          <span v-show="!supportIds[index].status" 
                class="filter-icon" 
                :style="{ color: '#' + item.icon_color, borderColor: '#' + item.icon_color}"
                >{{item.icon_name}}</span>
          <span :class="{ selecter_filter: supportIds[index].status}">{{item.name}}</span> 
        </li>
      </ul>
    </section>
    <section class="activity-btn">
      <button class="clear-btn" @click="clearAll()">清空</button>
      <button class="confirm-btn clear-btn" @click="confirmSelect()">
        确认
        <span v-show="this.filterNum!==0" style="color: #fff">({{this.filterNum}})</span>
        </button>
    </section>
  </section>
</transition>
```

点击配送方式和商家属性，根据相应索引和 id 计算选择项个数 filterNum，控制选择项文本颜色和图标

```js
selectDeliveryMode(id) {
  if (this.deliveryMode === null) {
    this.filterNum++
    this.deliveryMode = id
  } else if(this.deliveryMode == id) {
    this.filterNum--
    this.deliveryMode = null
  } else {
    this.deliveryMode = id
  }
},
selectSupportIds(index, id) {
  // status 为 true 时所选项图标变为 对号，为 false 时显示图标
  // 数组替换新的值
  this.supportIds.splice(index, 1, {
    status: !this.supportIds[index].status,
    id
  })
  // 遍历 重新计算filterNum
  this.filterNum = this.deliveryMode==null ? 0 : 1
  this.supportIds.forEach(item => {
    if (item.status) {
      this.filterNum++
    }
  })
}
```
点击清空按钮还原菜单栏，清空选项；点击确认按钮收起菜单栏，重新请求商铺。
```js
clearAll() {
  // 清空选择个数
  this.filterNum = 0
  // 配送方式清空
  this.deliveryMode = null
  // 商家属性 status 置为 false
  this.supportIds.forEach(item => {
    item.status = false
  })
},
confirmSelect() {
  // confirmStatus 状态转变时会重新进行数据请求
  this.confirmStatus = !this.confirmStatus
  // sortBy 为空时收起菜单
  this.sortBy = ''
}
```
在 ShopList.vue 中监听 confirmStatus 的变化，从而进行重新请求
```js
watch: {
  confirmSelect: function (value) {
    // confirmSelect 改变即调用 changeProp
    this.changeProp()
  }
}
methods: {
  async changeProp() {
    // 出现等待加载界面
    this.showLoading = true
    // 页面回到顶端
    this.offset = 0
    // 根据参数重新请求数据
    let res = await shopList(this.latitude, this.longitude, this.offset, '', this.restaurantCateIds, this.sortByType, this.deliveryMode, this.supportIds)
    // 考虑到本地模拟数据是引用类型，所以返回一个新的数组
    this.shopList = [...res]
    // console.log(this.shopList)
    if (res.length < 20) {
      this.touchend = true
    }
    // 隐藏等待加载界面
    this.hideLoading()
  }
}
```

# 目标功能

- [x] 定位功能
- [x] 选择城市
- [x] 搜索地址
- [x] 展示所选地址附近商家列表
- [x] 搜索美食
- [x] 根据距离、销量、评分、特色菜、配送方式等进行排序和筛选

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