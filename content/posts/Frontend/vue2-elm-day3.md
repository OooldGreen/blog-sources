---
title: "Vue2 Elm 项目记录 Day3"
date: 2022-11-22
lastmod: 2022-11-22
draft: false
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

# 使用 Swiper
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

# 上拉加载功能
上拉加载的实现就是在页面开始滑动时获取高度，在页面停止滑动后监听计算页面是否到达底部，触底则进行动作

- touchstart: 运动开始时获取元素
- touchmove: 监听 scrollTop 是否到达底部
- touchend: 运动结束时判断是否有惯性动作，惯性动作结束后判断是否到达底部

功能实现需要获取元素 padding 值和 margin 值，所以将用于获取 style 属性值的方法提取出来：
```js
export const getStyle = ((ele, attr, NumberNode='int') => {
  let target
  if (attr === 'scrollTop') {
    target = ele.scrollTop
  } else if (ele.currenStyle) {
    target = ele.currenStyle[attr]
  } else {
    target = document.defaultView.getComputedStyle(ele.null)[attr]
  }
  // 获取 opacity 需要获取小数
  return NumberNode == 'float' ? parseFloat(target) : parseInt(target)
})
```

上拉加载功能的实现：
```js
export const loadMore = {
  directives: {
    'load-more': {
      bind: (el, binding) => {
        let windowHeight = window.screen.height
        let height
        let setTop
        let paddingBottom
        let marginBottom
        let requestFram
        let oldScrollTop
        let scrollEl
        let heightEl
        let scrollType = el.attributes.type && el.attributes.type.value
        let scrollReduce = 2
        
        if (scrollType === 2) {
          scrollEl = el
          heightEl = el.children[0]
        } else {
          scrollEl = document.body
          heightEl = el
        }

        // 运动开始时获取元素：高度、offsetTop、padding、margin
        el.addEventListener('touchstart', () => {
          height = heightEl.clientHeight
          if (scrollType === 2) {
            height = height
          }
          setTop = el.offsetTop
          paddingBottom = getStyle(el, 'paddingBottom')
          marginBottom = getStyle(el, 'marginBottom')
        }, false)

        // 监听 scrollTop 的值是否到达底部
        el.addEventListener('touchmove', () => {
          loadMore()
        }, false)

        // 运动结束时判断是否有惯性动作，惯性动作结束后判断是否到达底部
        el.addEventListener('touchend', () => {
          oldScrollTop = scrollEl.scrollTop
          moveEnd()
        }, false)

        const moveEnd = () => {
          requestFram = requestAnimationFrame(() => {
            if (scrollEl.scrollTop !== oldScrollTop) {
              oldScrollTop = document.body.scrollTop
              loadMore()
              moveEnd()
            } else {
              cancelAnimationFrame(requestFram)
              // 为了防止鼠标抬起时已经渲染好数据从而导致重获取数据，重新获取dom高度
              height = heightEl.clientHeight
              loadMore()
            }
          })
        }

        // 判断是否到达底部
        const loadMore = () => {
          if (document.body.scrollTop + windowHeight >= height + setTop + paddingBottom + marginBottom - scrollReduce) {
            binding.value()
          }
        }
      }
    }
  }
}
```

在组件中使用：
```vue
<template>
  <div>
    <ul v-load-more="loaderMore">
      <li>...</li>
    </ul>
    <p class="touchend" v-if="touchend">没有更多了</p>
  </div>
</template>

<script>
  import { loadMore } from '../../../config/mUtils'
  export default {
  data() {
    return {
      offset: 0,
      shopList: [],
      touchend: false, // 没有更多数据
      showLoading: true, // 显示加载动画
      preventRepeatRequest: false, // 防止重复加载
    }
  },
  mixins: [loadMore],
  methods: {
    async loaderMore () {
      if (this.touchend) {
        return
      }
      if (this.preventRepeatRequest) {
        return
      }
      // 展示loading
      this.showLoading = true
      this.preventRepeatRequest = true
      // 再加载20条数据
      this.offset += 20
      let res = await shopList(this.latitude, this.longitude, this.offset, this.restaurantCategoryId)
      // 隐藏loading
      this.hideLoading()
      // 扩展shopList
      this.shopList = [...this.shopList, ...res]
      // 获取数据小于2-，表明没有更多数据了
      if(res.length < 20) {
        this.touchend = true
      }
      this.preventRepeatRequest = false
    }
</script>
```

# 返回顶部功能
返回顶部函数，只要文本高度大于 500，显示返回顶部按钮（在 callback 中自定义）
```js
const showBackFun = () => {
  if (document.body.scrollTop > 500) {
    cb(true)
  } else {
    cb(false)
  }
}
```

完整代码是在开始、结束、运动都对页面所在的位置进行判断决定是否显示按钮
```js
export const showBack = cb => {
  let requestFram
  let oldScrollTop

  document.addEventListener('scroll', () => {
    showBackFun()
  }, false)

  document.addEventListener('touchstart', () => {
    showBackFun()
  }, false)

  document.addEventListener('touchmove', () => {
    showBackFun()
  }, { passive: true })
  
  document.addEventListener('touchend', () => {
    oldScrollTop = document.body.scrollTop
    moveEnd()
  }, { passive: true })

  const moveEnd = () => {
    requestFram = requestAnimationFrame(() => {
      if (document.body.scrollTop !== oldScrollTop) {
        oldScrollTop = document.body.scrollTop
        moveEnd()
      } else {
        cancelAnimationFrame(requestFram)
      }
    })
    showBackFun()
  }

  const showBackFun = () => {
    if (document.body.scrollTop > 500) {
      cb(true)
    } else {
      cb(false)
    }
  }
}
```

在需要使用的组件的 mounted 中挂载
```js
import { showBack } from '../../../config/mUtils'

export default {
  data: {
    showBackTop: false // 显示返回顶部按钮
  },
  mounted: {
    showBack(status => {
      this.showBackTop = status
    })
  }
}
```
创建返回顶部 svg，引入，添加点击返回顶部事件
```vue
<template>
  <!-- 返回顶部 -->
  <aside class="back-top" @click="backTop" v-if="showBackTop">
    <svg class="back_top_svg">
      <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#backtop"></use>
    </svg>
  </aside>
</template>

<script>
  export default {
    methods: {
      backTop() {
        animate(document.body, {scrollTop: '0'}, 400, 'ease-out')
      }
    }
  }
</script>
```

# 加载动画
创建单独的 Loading 子组件
## 制作动画
一跳一跳的 load 动画
```css
@keyframes load {
  0%   { transform: translateY(0); }
  50%  { transform: translateY(-50px); } 
  100% { transform: translateY(0px); }
}
```

下方椭圆阴影变化动画（水果跳上去阴影变小，水果落下来阴影变大）
```css
@keyframes ellipse {
  0%   { transform: scale(1); }
  50%  { transform: scale(0.3); }
  100% { transform: scale(1); }
}
```

animation 语法
```css
animation: name duration timing-function delay iteration-count direction fill-mode play-state;
```
name 是设定的关键帧的名字；duration 是多久完成这个动画；timing-function 是动画运动的方式，比方说 linear 就是线性运动，ease-in-out 是动画缓慢过渡，加速再减速；iteration-count 设置为 infinite 迭代次数无穷大，即为不断重复。

设定背景图片为加载动画图，并配上动画：
```css
.load-img {
  background: url(../../../images/icon_loading.png) no-repeat 0 0;
  animation: load .6s ease-in-out infinite;
}

.load-ellipse {
  animation: ellipse .6s ease-in-out infinite;
}
```

## 图片定时切换
load 图片一列七个，每次弹跳变换不同的图片，绑定一个变换的 style 属性，通过调整 background 的位置改变图片
```html
<div class="load-img" :style="{ backgroundPositionY: -(positionY%7) + 'rem' }"></div>
```
再开一个定时器，每 0.6s 换一次图片。一定记得销毁组件之前销毁定时器。
```js
export default {
  data() {
    return {
      positionY: 0,
      timer: null
    }
  },
  mounted() {
    this.timer = setInterval(() => {
        this.positionY++;
    }, 600)
  },
  beforeDestroy() {
    clearInterval(this.timer)
  }
}
```

- [background 参考](https://developer.mozilla.org/en-US/docs/Web/CSS/background)
- [animation 参考](https://developer.mozilla.org/en-US/docs/Web/CSS/animation)

## 使用
引入 Loading 子组件
```vue
<template>
  <!-- 加载动画 -->
  <transition v-if="showLoading">
    <Loading/>
  </transition>
</template>

<script>
  import Loading from '../fun/Loading.vue'

  export default {
    data() {
      return {
        showLoading: true, // 显示加载动画，默认显示，页面加载后隐藏
      }
    },
    components: { Loading },
    mounted {
      shopList(this.latitude, this.longitude, this.offset, this.restaurantCategoryId).then(res => {
        this.hideLoading()
      })
    },
    methods: {
      hideLoading() {
        this.showLoading = false
      }
    }
  }
</script>
```

# 星星打分
新建一个组件，先做一排五个灰色星星
```vue
<template>
  <div class="container">
    <span class="star-container">
        <svg class="star" v-for="item in 5" :key="item">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#star"></use>
        </svg>
    </span>
  </div>
</template>

<style lang='scss' scoped>
@import '../../../style/mixin.scss';
.container {
  position: relative;
  width: 3rem;
}
.star-container {
  display: flex;
  position: absolute;
  height: 0.6rem;
  top: 0.2rem;
}
.star {
  height: 0.6rem;
  width: 0.6rem;
  fill: #d1d1d1;
}
</style>
```

再做一排橙色星星，宽度按照父组件传过来的 rating 计算比例
```vue
<template>
  <div :style="{ width: (rating/5)*3 + 'rem' }" class="star-overflow">
    <span class="star-container">
      <svg class="star orange" v-for="item in 5" :key="item">
          <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#star"></use>
      </svg>
    </span>
  </div>
</template>

<script>
export default {
  props: ['rating']
}
</script>

<style lang='scss' scoped>
.star-overflow {
  overflow: hidden;
  height: 100%;
  position: relative;
  .orange {
    fill: $orange;
  }
}
</style>
```
在父组件中引入，使用，传入 rating：
```vue
<template>
  <RatingStar :rating='item.rating'></RatingStar>
</template>

<script>
import RatingStar from '../fun/RatingStar.vue'
export default {
  components: { RatingStar }
}
</script>
```


# 目标功能

- [x] 定位功能
- [x] 选择城市
- [x] 搜索地址
- [x] 展示所选地址附近商家列表

<!-- - [ ] 搜索美食
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