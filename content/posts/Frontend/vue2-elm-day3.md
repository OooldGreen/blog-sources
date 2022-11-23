---
title: "Vue2 Elm 项目记录 Day3"
date: 2022-11-22
lastmod: 2022-11-22
draft: true
keywords: ["vue", "elm", "项目记录"]
description: ""
tags: ["vue2", "项目记录"]
categories: ["vue", "项目记录"]
author: ""
---
第三天啦，加油！

# 数据处理
需要处理数据将数组分成 8 个一组，方便后面引入 Swiper 的分页器效果
```js
msiteFood(geohash).then(res => {
  while (res.length !== 0) {
    this.foodCate.push(res.splice(0, 8))
  }
})
```

处理前返回的结果:
![处理前返回的结果](/image/frontend/vue-elm/food-category.png)

处理后的数组:
![处理后的数组](/image/frontend/vue-elm/food-category2.png)

#  使用 Swiper
- 在 Vue 中使用[Swiper](https://swiperjs.com/vue)  
- [Swiper 文档](https://www.swiper.com.cn/api)

先安装 Swiper
```shell
npm install swiper --save-dev
```

根据文档在组件中引入 Swiper 的 js 和 css，引入 Swiper 只有基础模块，如果需要其他组件功能需要另外引用，最后在 mounted 中初始化。
```js
// msite.vue

// 引入 js
import Swiper, { Pagination } from 'swiper'
// 引入 css
import 'swiper/css'
import 'swiper/css/pagination'

export default {
  components: { Swiper, SwiperSlide },
  mounted() {
    // 引入 Swiper
    const swiper = new Swiper('.swiper', {
      modules: [Pagination]
    })
  }
}
```
调试很久，报错 `swiper is not a constructor`，看到[另一篇博客](https://segmentfault.com/a/1190000020754838?utm_source=sf-similar-article)也有说这种报错，解决方法就是直接本地引入 js 和 css：
```js
import 'src/plugins/swiper.min.js'
import 'src/style/swiper.min.css'
```
但即使不再报错，我的 Pagination 还是没有显示，试了一下分页功能可以正常拖动，检查 `swiper-pagination` 中没有小圆点元素。检查我的 Swiper 版本：
![swiper 版本](/image/frontend/vue-elm/swiper.png)

所以直到把初始化 Swiper 的写法改成下面这样，小圆点才出来，无语住了
```js
new Swiper('.swiper-container', {
  pagination: '.swiper-pagination'
})
```

#

# 目标功能

- [x] 定位功能
- [x] 选择城市
- [x] 搜索地址
- [ ] 展示所选地址附近商家列表
- [ ] 搜索美食
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