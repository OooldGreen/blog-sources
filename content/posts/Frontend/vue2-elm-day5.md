---
title: "Vue2 Elm Day5" #标题
date: 2023-02-13T20:17:33+08:00 #创建时间
lastmod: 2023-02-13T20:17:33+08:00 #更新时间
categories: ["vue", "项目记录", "CS"]
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


## 点击‘...’出现提示气泡效果
气泡效果如下图：
![气泡效果](/image/frontend/vue-elm/bubble-css.png)

制作方法是内容为一个矩形盒子，右上角小三角是加伪元素 `::after`，做一个正方形旋转45度再定位得来。代码如下（样式用scss写的）：
```html
<header class="menu-list-header">
    <span class="menu-header-right" @click="showMenuDes(index)">...</span>
    <div class="menu-header-des-tip" v-if="showMenuDesIndex == index">
        <span>{{item.name}}</span>
        {{item.description}}
    </div>
</header>
```
```scss
.menu-list-header {
    .menu-header-right {
        color: #666;
        padding-right: 0.5rem;
    }
    .menu-header-des-tip {
        position: absolute;
        top: 1.7rem;
        right: 0.5rem;
        width: 65%;
        background-color: #39373a;
        opacity: 0.9;
        font-size: 0.7rem;
        color: #fff;
        border-radius: 0.3rem;
        padding: 0.6rem 0.5rem 0.6rem;
        span {
            color: #fff;
            font-size: 0.8rem;
            font-weight: bold;
            margin-right: 0.3rem;
        }
        &::after {
            content: '';
            position: absolute;
            top: -0.2rem;
            right: 0.18rem;
            background-color: #39373a;
            width: 0.7rem;
            height: 0.7rem;
            transform: rotate(45deg);
        }
    }
}
```

js代码如下：
```js
data() {
    return {
        showMenuDesIndex: null, // 显示菜单标题描述
    }
},
methods: {
    showMenuDes(index) {
        if(this.showMenuDesIndex == index) {
            this.showMenuDesIndex = null
        } else {
            this.showMenuDesIndex = index
        }
    }
}
```

后面“新品”标识也可以这样做，将正方形方块旋转45度定位到商品盒子右上角，隐藏溢出即可

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