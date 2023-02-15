---
title: "Vue2 Elm Day5" #标题
date: 2023-02-13T20:17:33+08:00 #创建时间
lastmod: 2023-02-13T20:17:33+08:00 #更新时间
categories: ["vue", "项目记录"]
tags: ["vue2", "项目记录"]
description: "" #描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
# draft: false # 是否为草稿
# comments: true #是否展示评论
# showToc: true # 显示目录
# TocOpen: true # 自动展开目录
# hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
# disableShare: true # 底部不显示分享栏
# showbreadcrumbs: true #顶部显示当前路径
# cover:
#     image: "" #图片路径：posts/tech/文章1/picture.png
#     caption: "" #图片底部描述
#     alt: ""
#     relative: false
---
# 餐馆列表页
首先在 ShopList.vue 中将单纯的列表改为路由跳转
```html
<ul v-load-more="loaderMore" type="1" v-if="shopList.length">
    <router-link :to="{ path: 'shop', query: { geohash, id: item.id} }" class="shop-info" v-for="item in shopList" :key="item.id"></router-link>
</ul>
```

## 父子页面跳转，父页面隐藏
路由 index.js ，添加 showPage 控制页面的显示与隐藏：
```js
const routes = [
    {
    path: '/shop',
    component: Shop,
    children: [
      {
        path: 'shopDetail',
        component: ShopDetail,
        meta: {
          showPage: false
        }
      }
    ],
    meta: {
      showPage: true
    }
  }
]
```

父页面 Shop.vue 中需要添加 router-view 显示子页面
```html
<!-- 父页面 -->
<!-- 添加 v-show 控制隐藏 -->
<section v-show="$route.meta.showPage"></section>
<!-- 子路由显示的区域 -->
<!-- 后面要给页面加进入动画，所以用 transition 包裹 -->
<transition>
    <router-view></router-view>
</transition>
```

## 子页面显示动画

### 页面从右侧进入动画
```js
<transition name="router-slide" mode="out-in">
    <router-view/>
</transition>
```
在 style 中添加动画
```css
.router-slide-enter-active,
.router-slide-leave-active {
  transition: all .3s;
  transform: translateX(0);
  opacity: 0;
}
.router-slide-enter,
.router-slide-leave-to {
  transform: translateX(100%); 
}
```

### 渐变消失出现动画
```html
<transition name="fade">
    <section  class="license-img-container"></section>
</transition>
```
在 style 中添加动画
```css
.fade-enter-active,
.fade-leave-active {
  transition: all .3s;
  opacity: 1;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
.license-img-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, .5);
}
```

# 食品详情页面

# 目标功能

- [x] 定位功能
- [x] 选择城市
- [x] 搜索地址
- [x] 展示所选地址附近商家列表
- [x] 搜索美食
- [x] 根据距离、销量、评分、特色菜、配送方式等进行排序和筛选
- [ ] 餐馆视频列表页
- [ ] 单个食品详情页面

<!-- - [ ] 购物车功能
- [ ] 店铺评价页面

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