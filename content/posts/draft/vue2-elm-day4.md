---
title: "Vue2 Elm 项目记录 Day4"
date: 2022-11-28
lastmod: 2022-11-28
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