---
title: "期末知识点复习"
date: 2022-10-30T10:35:16+08:00
lastmod: 2022-11-06
draft: false

keywords: ["interview"]
description: ""
tags: ["interview", "js", "Vue", "CS"]
categories: ["Frontend"]
author: ""
math: true

# hidemeta: false
# comments: true
# canonicalURL: "https://canonical.url/to/page"
# disableShare: false
# disableHLJS: false
# hideSummary: false
# searchHidden: false
# ShowReadingTime: true
# ShowBreadCrumbs: true
# ShowPostNavLinks: true
# ShowWordCount: true
# ShowRssButtonInSectionTermList: true
# UseHugoToc: false
---

方便反复诵读复习
<!--more-->

# 1. 计算机原理

## 1.1 输入 URL 到渲染完成的整个过程
1. 根据域名递归查找地址，先查找缓存，没有则进行 DNS 解析，向域名服务器获取 IP ，返回给浏览器
2. 建立 TCP 连接
3. 客户端发送请求，服务端收到请求并响应
4. 客户端接收到响应结果，得到资源文档，可能进行解压缩
5. 解析文档构建 DOM 树和 CSSOM 树，合成 Render 树
6. 进行布局，再渲染，呈现页面

[面试题之从敲入 URL 到浏览器渲染完成](https://juejin.im/post/5b9ba9c15188255c8320fe27)

## 1.2 TCP 三次握手和四次挥手

[详解TCP 三次握手和四次挥手](https://blog.csdn.net/qq_31587111/article/details/122731287)

### TCP 三次握手
- 客户端发送 SYN 报文，初始化 ISN 序列，SYN 标识为置为一，发送后进入 SYN_SENT 状态
- 服务端接收后发送 SYN + ACK 报文，并把客户端发送报文的 ISN 序列号 +1 作为自己的 ACK 字段，把 ACK 和 SYN 标志位都置为 1，进入 SYN_REVD 状态
- 客户端接收后发送 ACK 报文，ACK 标志位置为 1，这次可以携带应用层数据，进入 ESTABLISHED 状态

### 为什么是三次握手？
- 确保序列号有效的同步
- 确定客户端和服务端收发正常
- 避免资源浪费

### TCP 四次挥手
- 客户端发送 FIN 报文，进入 FIN_WAIT_1 状态
- 服务端收到后，发送 ACK 报文，进入 CLOSED_WAIT 状态
- 客户端收到报文后，进入 FIN_WAIT_2 状态
- 服务端处理完数据后，发送 FIN 报文，服务端进入 LAST_ACK 状态
- 客户端收到后，发送 ACK 报文，进入 TIME_WAIT 状态，等待 2MSL 时间后自动关闭，进入 CLOSED 状态
- 服务端收到 ACK 报文后，直接进入 CLOSED 状态

### 为什么是四次挥手？
- 确保客户端和服务端都不再接收和发送数据
- 服务端可能有数据要处理，所以 ACK 和 FIN 分开发送
- 等待 2MSL 时间是为了确保服务端收到报文信息，如果没有收到，服务端会重新发送 FIN 报文，客户端就要再次发送 ACK 报文，因此至少是一个报文来回的时间

## 1.3 TCP 和 UDP 的区别
- TCP 面向连接传输，需要先建立连接，UDP 不需要
- TCP 基于可靠性传输，UDP 保证传输速度不保证传输质量
- TCP 一对一传输，UDP 可以一对一、一对多、多对多
- TCP 有拥塞控制和流量控制，UDP 没有
- UDP 头部长度固定，开销小

## 1.4 HTTP 和 HTTPS
### HTTP 和 HTTPS 的区别
- HTTPS 加入了 SSL 证书进行加密，HTTP 不用
- HTTPS 更安全，更利于 SEO
- HTTPS 基于传输层，HTTP 基于应用层
- HTTPS 标准端口 43，HTTP 标准端口 80

### [状态码](https://zh.m.wikipedia.org/zh-hans/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)
- 1: 临时响应，需要继续处理消息  
    102 Processing: 处理中，通常需要很长时间才能完成请求时发送
- 2: 请求成功  
- 3: 重定向  
    301 Moved Permanently: 

    302 Found: 

    304 Not Modified:
- 4: 请求失败，客户端错误  
    400 Bad Request: 

    404 Not Found: 
- 5: 请求失败，服务端错误  
    500 Internal Server Error: 

    502 Bad Gateway: 

## 1.5 用户登录认证
### Cookie 和 Session
在服务端创建 Session，将 SessionId 发送给客户端保存在 Cookie 中，在以后发送请求时自动携带，服务端分解 Cookie 后验证成功将请求返回客户端。

缺点：
- 数据都保存在服务器端，服务器有压力。
- 用 Cookie 不用 Session，所有数据都保存在客户端的话一旦 Cookie 被劫持，所有数据全部泄露

### Token
无状态令牌，用户信息被加密到 Token 中，服务端解密就能得知是哪个用户，一般保存在浏览器本地缓存中。服务器端生成 Token 后返回给客户端，客户端再次发送请求时在 headers 中添加 Token，服务端校验成功就会返回数据。

生成方式：JWT (Json Web Token)，三部分组成 header + payload + signature

优点：
- 防止 CSRF 攻击，token 可以存放在前端任何地方，不用保存在 cookie 中，浏览器也不会自动把 token 放入 headers 里，不能通过 document.cookie 直接拿到，攻击者无法访问用户的 token
- 服务器端不需要存放 token，减轻服务端压力

## 1.6 浏览器缓存
浏览器缓存将文件保存在客户端，可以减少对带宽的占用，减少响应时间，提升用户体验。

页面的缓存状态是由 http-header 字段决定的，相关的字段为 Cache-Control、Expires、Last-Modified/If-Modified-Since、Etag/If-None-Match。

大致分为协商缓存和强缓存。

### 强缓存
Cache-Control：
- 设置 max-age，设定时间内可以直接从本地缓存中读取文件并加载
- public/private，设置缓存能否在用户间共享
- no-cache， 不进行缓存

Expires：
设置页面过期的时间，过期时间内可以直接读取缓存

### 协商缓存
协商缓存有两组字段：Last-Modified/If-Modified-Since 和 Etag/If-None-Match，其中 Etag 优先级更高。

请求过程是检查是否命中协商缓存，如果资源无更新，返回 304 并直接读取资源，如果修改过，返回 200 并加载新资源。

- Last-Modified/If-Modified-Since：分别表示最后修改的时间和缓存对象最后修改时间，相互对比得出资源是否被更新
- Etag/If-None-Match：hash 生成特殊标识，如果更新资源 hash 值就会改变，再缓存文件存储的字符串进行对比

## 1.7 浏览器本地存储方式 cookie、localStorage 和 sessionStorage
cookie、localStorage 和 sessionStorage 都是浏览器本地缓存方式。

区别：
- cookie 在服务器和客户端之间来回传递，localStorage 和 sessionStorage 都是浏览器本地保存
- cookie 存储空间很小，大概只有 4kb ，sessionStorage 和 localStorage 存储空间较大，大概在 5M 左右
- sessionStorage 在浏览器窗口关闭时就会自动清空，localStorage 需要手动清除，cookie 在设置的有效期之内一直有效
- localStorage 和 cookie 在所有同源窗口中共享，sessionStorage 换个页面就不行了

## 1.8 安全攻击
### CSRF
跨站请求伪造。伪造 cookie ，通过模仿用户行为进行操作。

防御方式：
- 使用验证码
- 添加 referer 字段
- 使用 token 验证

### XSS
跨站脚本攻击。通过注入恶意代码，读取用户敏感信息，在用户浏览页面时进行攻击。

防御方式：
- 开启 set-cookie: httponly，禁止 js 脚本访问 cookie
- 输入输出检查，对用户输入的字符进行转义

## 1.9 前端优化
### SEO方法
- meta标签的优化：标题、网站描述、关键词、目录、编码语种等
- 关键词：密度、相关度、突出性考虑
- 了解主要搜索引擎的检索排序方式
- 链接交换，其他网站链接到你的网站越多，你的网站访问量就越大
- 合理使用标签，使搜索网站更容易抓取内容

### 性能优化
- 内容
    - 减少请求
    - 减少 DNS 查找
    - 加载顺序 CSS 在前，JS 在后
    - 减少 404
- 服务器优化
    - 使用 CDN 缓存
    - 延迟加载、预加载组件
    - GZip 压缩
    - 路由懒加载、图片懒加载
    - 使用雪碧图

#### 图片懒加载
[参考](https://juejin.cn/post/6961227083573886984)
- 内联图片：将图片的 src 放在自定义的属性里面，利用 js 检查标签是否在视窗中，如果在将自定义的属性赋值给 src 属性就可以了。
    ```js
    // 获取所有图片
    const imgList = document.querySelectorAll('img')
    // 用于记录当前显示到了哪一张图片
    let index = 0;
    function lazyload() {
        // 获取浏览器视口高度,这里写在函数内部是考虑浏览器窗口大小改变的情况
        const viewPortHeight = window.innerHeight
        for (let i = index; i < imgList.length; i++) {
            // 这里用可视区域高度减去图片顶部距离可视区域顶部的高度
            const distance = viewPortHeight - imgList[i].getBoundingClientRect().top;
            // 如果可视区域高度大于等于元素顶部距离可视区域顶部的高度，说明图片已经出现在了视口范围内
            if (distance >= 0) {
            // 给图片赋值真实的src，展示图片
            imgList[i].src = imgList[i].getAttribute('data-src');
            // 前i张图片已经加载完毕，下次从第i+1张开始检查是否需要显示
            index = i + 1;
            }
        }
    }

    // 再加一个节流函数
    // 使用的时候挂载： window.onload = lazyload()
    ```
- intersection observer：要考虑兼容性
- css图像：利用 background-image，当 js 检测到元素处于视窗中，加一个 class 类名引用外部图片资源

[更全面的 js 功能实现](https://juejin.cn/post/6844904066418491406#heading-3)

#### 路由懒加载
动态导入代替静态导入

```js
// 将
import UserDetails from './views/UserDetails'
// 替换成
const UserDetails = () => import('./views/UserDetails')
```

使用 webpack（babel）分块导入，只需要使用命名 chunk，魔法注释提供 chunk name 

```js
const Users = () => import(/* webpackChunkName: "users-rights-roles" */ '../components/user/Users.vue')
const Rights = () => import(/* webpackChunkName: "users-rights-roles" */ '../components/power/Rights.vue')
   ```

#### 服务端渲染 SSR
服务器端生成HTML直接返回给浏览器。

实现方式：
- Vue + Nuxt.js

优势：
1. 减少网络传输，提高传输效率
2. 首屏渲染快
3. 有利于 SEO，提高搜索效率

缺点：
1. 不利于前后端分离，开发效率不高
2. 占用服务器的资源

## 1.10 axios
- axios 是一个基于 promise 的 HTTP 库，支持 promise 的所有 API
- 它可以拦截请求和响应
- 它可以转换请求数据和响应数据，并对响应回来的内容自动转换为 json 类型的数据
- 它安全性更高，客户端支持防御 XSRF

#### axios 拦截器
拦截器分为请求拦截器和响应拦截器

请求拦截器用于在请求之前做处理，例如带上 token、时间戳等，响应拦截器用于接口返回之后做处理，例如判断 token 是否过期。

原理：创建一个 chn 数组，数组中保存了拦截器相应方法和 dispatchRequest，请求拦截器放到 dispatchRequest 前面，响应拦截器放到 dispatchRequest 后面，把请求拦截器和响应拦截器 forEach 放到数组中，用 promise 出队列的方式保证他们的执行顺序挨个执行。

## 1.11 跨域方法
同源策略：协议、域名、端口号都相同，保护浏览器免受攻击

- JSONP：利用 script 标签可以进行跨域请求的特性，创建一个 script 标签写入跨域请求，并通过 callback 接收响应。仅支持 get 方法
- CORS：后端处理，前端写入字段 `Access-Control-Allow-Origin`
- Vue Proxy
    ```js
    // vue.config.js/webpack.config.js 
    // 优点：配置简单 
    // 缺点：不能灵活控制请求是否走代理，因为都会走代理 
    module.exports={
        devServer:{
            proxy:'http://xxx.xxx.xxx:5000' 
        } 
    }
    ```
- WebSocket：提供全双工、双向通信
- PostMessage：HTML5 中的 API

## 1.12 前后端实时通信
- 轮询：客户端隔一段时间询问一次，适用于小型应用，实用性不高
- 长轮询：客户端询问后，服务端有新消息才返回
- WebSocket：长连接，适用于微信、网络互动游戏等
- iframe 流：插入一个隐藏的 iframe，利用其 src 属性在服务器和客户端之间创建一条长连接，服务器向 iframe 传输数据（通常是 HTML，内有负责插入信息的 javascript），来实时更新页面。兼容性好，但长连接会增加开销
- SSE：单向通道，服务器向浏览器推送资源

## 1.13 主流浏览器
| 浏览器 | 内核  |
|----|----|
| IE | Trident |
|Firefox|Gecko|
|Chrome|Blink|
|Safari|Webkit|
|Opera|Blink|

## 1.14 get 和 post 请求的区别
- get 回退是无害的，post 会再次提交请求
- get 参数通过 url 进行传递，更不安全，有长度限制；post 参数放在 request body 中，更安全，没有长度限制
- get 会被浏览器主动缓存保存参数，post 不会
- get 产生一个 TCP 数据包，post 产生两个（在 Firefox 中只有一个）

# 2. JS
## 2.1 数据类型
### 数据类型
Number, Object，String, Boolean, Null, Undefined, BigInt, Symbol

BigInt 和 Symbol 是 ES6 新的数据类型，Symbol 创建一个不重复的变量，可以用于 object 的 key。

- 引用数据类型：Object，存储在堆中，栈中一个指针指向它
- 普通数据类型：除了 Object 以外其他的数据类型，存储在栈中

### [null 和 undefined 的区别](https://www.nowcoder.com/exam/interview/detail?questionClassifyId=0&questionId=2412346&questionJobId=156&type=1)
- null：表示被定义了，但是是空值，占用内存，通过 typeof 判断是 object (这是因为 typeof 通过二进制前三位判断数据类型，null 和 object 一样前三位都是 `000`)
- undefined：是一个变量未赋值 或 一个函数没有返回值 或 访问一个对象不存在 或 一个函数定义了形参但没有传递实参，通过 typeof 判断是 undefined

## 2.2 方法
### 判断数据类型的方法
1. typeof：只能区分基本数据类型，引用数据类型除了 function 会返回 function，其余都是 object
2. constructor：用于引用数据类型，检测方法是获取实例的构造函数判断和某个类是否相同，如果相同就说明该数据是符合那个数据类型的，这种方法不会把原型链上的其他类也加入进来，避免了原型链的干扰。
3. instanceof：用于引用数据类型，检测数据类型是否在当前变量的原型链上
4. Object.prototype.toString.call()：适用于所有数据类型

### 数组方法
增删改查：push、pop、shift、unshift、concat、join、reverse、sort、map、forEach、reduce、slice、splice、forEach、filter、indexOf

改变原数组：push、pop、shift、unshift、reverse、sort、splice

不改变原数组：concat、join、filter、slice、reduce、map

#### 数组去重
1. `Array from(new Set(array))`
2. `[...new Set(array)]`
3. filter
4. 采用对象数组方法去重
    ```javascript
    function sort(arr){
        let obj = {};
        let newArr = [];
        for(let i = 0; i < arr.length; i++){
            if(!obj[arr[i]]){
                obj[arr[i]] = 1;
                newArr.push(arr[i]);
            }
        }
        return newArr;
    }
    ```

#### 翻转字符串
split -> reverse -> join

```js
function reverse(str){
    for(let i = 0; i < str.length; i++){
        return str.split('').reverse().join('');
    }
}
```

### map 和 forEach
- map 不改变原数组，开辟新空间，返回新数组，处理速度快
- forEach 返回 undefined，改变原数组

### for...of 和 for...in
- for in 通用的对象遍历，遍历对象的 key，也包括原型链上的属性的遍历，是以任意顺序迭代对象的可枚举属性
- for of 遍历的是值，遍历可迭代属性，不能直接用来遍历对象

```javascript
Object.prototype.objCustom = function() {}; 
Array.prototype.arrCustom = function() {};

let iterable = [3, 5, 7];
iterable.foo = 'hello';

for (let i in iterable) {
  console.log(i); // 0, 1, 2, "foo", "arrCustom", "objCustom"
}

for (let i in iterable) {
  if (iterable.hasOwnProperty(i)) {
    console.log(i); // 0, 1, 2, "foo"
  }
}

for (let i of iterable) {
  console.log(i); // logs 3, 5, 7
}
```
### ...扩展符
用于取出对象中可遍历的属性，浅拷贝到当前对象中

## 2.3 异步方法
- 回调函数
- Promise
- Generator：一个状态机，yield 暂停，next 下一步
- Async/Await：基于 Promise 实现，看起来很像同步代码，使用上清晰明了

### [Promise]()
三种状态：PENDING、FULFILLED、REJECTED，状态一旦改变就不会再改变了

方法:
- Promise.resolve(value)
- Promise.reject(reason)
- Promise.then：属于微任务
- Promise.all：全部 Promise 成功，返回成功
- Promise.any：一个 Promise 成功，返回成功
- Promise.race：返回处理最快的那个的返回值，无论成功还是失败

#### 实现 Promise

```js
// 三个常量用于表示状态
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

function MyPromise(fn) {
    const that = this
    this.state = PENDING

    // value 变量用于保存 resolve 或者 reject 中传入的值
    this.value = null

    // 用于保存 then 中的回调，因为当执行完 Promise 时状态可能还是等待中，这时候应该把 then 中的回调保存起来用于状态改变时使用
    that.resolvedCallbacks = []
    that.rejectedCallbacks = []


    function resolve(value) {
         // 首先两个函数都得判断当前状态是否为等待中
        if(that.state === PENDING) {
            that.state = RESOLVED
            that.value = value

            // 遍历回调数组并执行
            that.resolvedCallbacks.map(cb=>cb(that.value))
        }
    }
    function reject(value) {
        if(that.state === PENDING) {
            that.state = REJECTED
            that.value = value
            that.rejectedCallbacks.map(cb=>cb(that.value))
        }
    }

    // 完成以上两个函数以后，我们就该实现如何执行 Promise 中传入的函数了
    try {
        fn(resolve,reject)
    }catch(e){
        reject(e)
    }
}
```

#### 实现 Promise.then
```js
// 最后我们来实现较为复杂的 then 函数
MyPromise.prototype.then = function(onFulfilled,onRejected){
  const that = this

  // 判断两个参数是否为函数类型，因为这两个参数是可选参数
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v;
  onRejected = typeof onRejected === 'function' ? onRejected : e => throw e;

  // 当状态不是等待态时，就去执行相对应的函数。如果状态是等待态的话，就往回调函数中 push 函数
  if(this.state === PENDING) {
      this.resolvedCallbacks.push(onFulfilled)
      this.rejectedCallbacks.push(onRejected)
  }
  if(this.state === RESOLVED) {
      onFulfilled(that.value)
  }
  if(this.state === REJECTED) {
      onRejected(that.value)
  }
}
```

#### 实现 Promise.all
```js
// 实现Promise.all()对象
function promiseAll(promises) {
  return new Promise(function(resolve, reject) {
    if (!isArray(promises)) {
      return reject(new TypeError('arguments must be an array'));
    }
    var resolvedCounter = 0;
    var promiseNum = promises.length;
    var resolvedValues = new Array(promiseNum);
    for (var i = 0; i < promiseNum; i++) {
      (function(i) {
        Promise.resolve(promises[i]).then(function(value) {
          resolvedCounter++
          resolvedValues[i] = value
          if (resolvedCounter == promiseNum) {
            return resolve(resolvedValues)
          }
        }, function(reason) {
          return reject(reason)
        })
      })(i)
    }
  })
}
```

## 2.4 原型及原型链
每个构造函数都有一个原型对象 prototype，原型有一个属性 constructor 指回构造函数，实例有一个内部指针 `__proto__` 指向原型 prototype。每个构造函数都有一个属性 prototype 指向原型对象，原型对象又有指针指向上一级原型对象，构成一条原型链，原型链的顶端是 null

{{< figure src="/image/frontend/原型和原型链.png" title="实例、构造函数和原型的关系">}}


## 2.5 ES6 新特性
- let 和 const
- 箭头函数
- 模块化导入导出 import/export
- 异步 promise
- 解构赋值

### 类
新的语法糖结构

```js
class Person {
    constructor() {
        console.log('Person')
    }
}

let p = new Person; // Person
```

### 箭头函数
- 没有 this，继承上下文的 this
- 不能 new，没有 arguments

### let、const 和 var 的区别
- var 有变量提升，let、const 没有变量提升
- var 是函数作用域，let 和 const 是块级作用域
- var 能声明多次，let 和 const 只能声明一次
- const 变量声明必须赋值，一旦赋值不可更改

## 2.6 New
- 创造一个空对象
- 添加属性 `__proto__` 指向构造函数的原型对象
- 创建 this 上下文
- 返回这个函数

## 2.7 闭包
在子函数内部调用了父函数的变量，并在函数外被引用就形成闭包。

闭包中的变量在函数运行完毕后也不会被销毁，因此如果不及时回收可能造成内存泄露。

## 2.8 事件循环
从宏任务 script 开始，按照 `宏任务 -> 微任务` 的顺序循环处理。先从上到下执行，遇到宏任务放入宏任务队列，遇到微任务放到微任务队列，执行完进入微任务队列出队列处理，然后再处理宏任务，交替进行。

- 宏任务 (macrotask)：script、定时器（setTimeout、setTimeInterval）、I/O
- 微任务 (microtask)：promise 的回调、nextTick、mutationObserver

## 2.9 事件委托
利用事件冒泡，把事件绑定到父元素上，当子元素发生事件时，通过冒泡触发父元素身上的事件从而做出响应。

可以避免给每个子元素绑定事件，代码冗余且要加入子元素还要再次绑定事件。

## 2.10 深拷贝和浅拷贝
浅拷贝只拷贝栈中的内容，深拷贝拷贝堆和栈中的内容。

深拷贝的方法：
- 第三方库，例如 lodash 中的 `cloneDeep()` 方法
- `JSON.parse(JSON.stringify(obj))`，对于有些数据类型拷贝之后可能会丢失数据
- 新的方法 `structuredClone()`，但是对于 error 和 DOM 对象无法拷贝
- 自己写，递归拷贝
    ```js
    function deepClone(obj) {
        // 检测是否为 Array
        let objClone = Array.isArray(obj) ? [] : {};
        if (obj && typeof obj === "object") {
            for (key in obj) {
                if (obj.hasOwnProperty(key)) {
                    //判断obj子元素是否为对象，如果是，递归复制
                    if (obj[key] && typeof obj[key] === "object") {
                        objClone[key] = deepClone(obj[key]);
                    } else {
                        //如果不是，简单复制
                        objClone[key] = obj[key];
                    }
                }
            }
        }
        return objClone;
    }
    ```

## 2.11 bind, apply 和 call
- 作用都是改变 this 指向
- apply 接收参数只能是一个数组，call 和 bind 接收参数要一个一个传入参数
- bind 直接改变 this 指向并返回一个新函数，之后再调用 this 都指向 bind 绑定的第一个参数

### 实现 apply
```js
Function.prototype.myApply = function(context, args) {
    // 默认上下文为 window，不传参数就是空数组
    context = context || windows;
    args = args ? args : [];

    // 给 context 新增一个独一无二的属性
    // 用隐式绑定的方法调用函数
    // 再删除添加的属性
    const key = Symbol();
    context[key] = this;
    const result = context[key](...args);
    delete context[key];

    return result; 
}
```

### 实现 bind
```js
Function.prototype.myBind = function(context, ...args) {
    args = args ? args : []; 
    const fn = this;

    const result = function(..fnArgs) {
        // 如果是通过 new 调用的，绑定 this 为实例对象
        if (this instanceof result) {
            fn.apply(this, [...args, ...fnArgs]);
        } else { // 否则普通函数形式绑定 context
            fn.apply(context, [...args, ...fnArgs]);
        }
    }

    // 绑定原型链
    result.prototype = Object.create(fn.prototype);

    return result;
}
```

## 2.12 垃圾回收

### 引用计数
对一个对象被引用的次数计数，引用计数为 0 则清除

可以即刻回收，计算过于复杂，效率低

### 标记清除
对所有活动的对象进行标记，清除阶段没有标记的会被清除

计算简单，但内存空间不连续，造成内存碎片

## 2.13 防抖节流

### 防抖
只认最后一次，n 秒之内又被触发则重新计时，例如按钮提交

```js
// fn  是我们需要包装的事件回调, delay 是每次推迟执行的等待时间
function debounce(fn, delay) {
  // 定时器
  let timer = null
  
  // 将 debounce 处理结果当作函数返回
  return function () {
    // 保留调用时的 this 上下文
    let context = this
    // 保留调用时传入的参数
    let args = arguments

    // 每次事件被触发时，都去清除之前的旧定时器
    if(timer) {
        clearTimeout(timer)
    }
    // 设立新定时器
    timer = setTimeout(function () {
      fn.apply(context, args)
    }, delay)
  }
}

// 用 debounce 来包装 scroll 的回调
const better_scroll = debounce(() => console.log('触发了滚动事件'), 1000)

document.addEventListener('scroll', better_scroll)
```

### 节流
只认第一次，在一定时间内只能触发一次

```js
// fn 是我们需要包装的事件回调, delay 是时间间隔的阈值
function throttle(fn, delay) {
  // last 为上一次触发回调的时间
  let last = 0
  
  // 将 throttle 处理结果当作函数返回
  return function () {
      // 保留调用时的 this 上下文
      let context = this
      // 保留调用时传入的参数
      let args = arguments
      // 记录本次触发回调的时间（获取时间戳）
      let now = +new Date()
      
      // 判断上次触发的时间和本次触发的时间差是否小于时间间隔的阈值
      if (now - last >= delay) {
          // 如果时间间隔大于我们设定的时间间隔阈值，则执行回调
          last = now;
          fn.apply(context, args);
      }
    }
}

// 用 throttle 来包装 scroll 的回调
const better_scroll = throttle(() => console.log('触发了滚动事件'), 1000)

document.addEventListener('scroll', better_scroll)
```

## 2.14 函数柯里化
接收的多个参数转化为一个一个的参数的函数

## 2.15 请求方式
### fetch 请求

### ajax 请求

## 2.16 继承
- 原型链继承

- 组合继承

- 寄生式继承

- 寄生式组合继承

## 2.17 设计模式
### 创建型
- 单例模式
- 建造者模式

### 结构型
- 代理模式
- 装饰器模式
- 适配器模式

### 行为型
- 发布订阅模式
- 迭代器模式

# 3. HTML 和 CSS
## 3.1 HTML5新特性
### 增加的特性
- canvas
- video 和 audio
- localStorage 和 sessionStorage
- 语义化更好的元素内容，如 article, footer, header, nav, svg, figure, menu
- 新的技术 webworker、 websocket

### 移除的元素
- 纯表现的元素，例如 big，basefont，s，u
- 对可用性产生负面影响，例如 frame，noframes

## 3.2 HTML 语义化
- 结构清晰，利于 SEO
- 便于维护
- 没有 css 也能看懂
- 便于对浏览器、搜索引擎解析

## 3.3 HTML 中 title 属性和 alt 属性的区别
- title 是图片加载后的标题，鼠标放上去后会显示
- alt 是图片没有加载出来时候显示的内容

## 3.4 重绘和重排
### 重绘 (repaint)
部分重新绘制，style 修改。重绘不一定发生重排。

触发场景：
- color 修改
- text-align 修改
- a:hover 修改

### 重排/回流 (reflow)
重新排版布局，涉及 DOM 的排版布局问题，性能损耗更多。重排必定发生重绘。

触发场景：
- 盒子长宽的改变
- 动画，伪类等引起的元素表面改动
- display: none
- scroll、resize 页面
- background 的修改
- appendChild 等 DOM 元素操作
- 读取元素的属性，如 offsetLeft、offsetTop、scrollTop/Left/...

### 如何避免重绘重排
- 通过改变 class 批量改变 style，同时尽可能减少受影响的 DOM
- 使用 absolute 或 fixed 脱离文档流
- GPU 加速，开启之后的元素会被独立出来，不再影响其他布局
- 避免用 table 进行布局

## 3.5 BFC
块级格式化上下文，是一块独立渲染的区域，内部元素的渲染不会影响外部的元素。BFC 布局会把盒子在垂直方向上一个一个排列，盒子之间的距离由 margin 决定，两个相邻盒子的 margin 会互相重叠。BFC 区域不会与 float box 重叠，计算高度时，浮动盒子也参与计算。

触发 BFC
- 根元素 html
- display 为 inline-block，table-cell, table-caption
- position 为 fixed 或 absolute
- float 不为 none
- oveflow 不为 visible

## 3.6 CSS3 新特性
- border-radius，box-shadow
- 文本效果 text-shadow、font-family、font-weight、font-style
- 多背景 rgba
- 动画 @keyframes
- 线性渐变
- 新增伪类

## 3.7 盒子模型
- 标准盒模型：`box-sizing: content-box`，设置的宽高只表示内容的宽高，不包含 padding 和 border
- 怪异盒模型：`box-sizing: border-box`，设置的宽高就是整个盒子的大小，包含 padding 和 border

## 3.8 水平垂直居中
1. 绝对定位
    ```css
    .item{
        position: absolute;
        top: 50%;
        left: 50%;
        margin-top: -75px;  /* 设置margin-left / margin-top 为自身高度的一半 */
        margin-left: -75px;
    }
    ```
2. transform
    ```css
    .item{
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);  /* 使用 css3 的 transform 来实现 */
    }
    ```
3. flex 布局
    ```css
    .parent{
        display: flex;
        justify-content: center;
        align-items: center;
    }
    ```
4. grid 布局
    ```css
    .parent {
        display: grid;
        justify-content: center;
        align-items: center;
     }
    ```
5. table-cell 布局
    ```css
    .parent {
        display: table-cell;
        text-align: center;
        vertical-align:center;
    }
    .item {
        display: inline-block;
    }
    ```

## 3.9 画一个三角形
盒子宽高都设为 0，只设置 border，三条边都设置 transparent，只留一条边有颜色

```css
#demo {
 width: 0;
 height: 0;
 border-width: 20px;
 border-style: solid;
 border-color: transparent transparent red transparent;
}
```

## 3.10 三栏布局
三栏布局的要求是两边宽度固定，中间的盒子自适应布局，高度随内容撑起。

### 圣杯布局
三个盒子放在一个父元素节点中，设置父盒子 padding 为左右两栏的宽度且 `overflow: hidden`，三个盒子设置左浮动 `float: left`。中间盒子宽度设置 100%，两栏宽度设好，左栏 margin 设置 -100% 且右移一个自身宽度，右栏 margin 设置 -自身宽度，且左移一个自身宽度。

好处是不用添加 DOM 节点，但是 middle 部分小于 left 部分时布局会破碎。

```html
<style>
    .parent {
        padding: 0 200px;
        overflow: hidden;
    }
    .column {
        float: left;
        position: relative;
        height: 200px;
    }
    .middle {
        width: 100%;
        background-color: pink;
    }
    .left {
        width: 200px;
        background-color: #eee;
        margin-left: -100%;
        left: -200px;
    }
    .right {
        width: 200px;
        background-color: #eee;
        margin-left: -200px;
        left: 200px;
    }
</style>

<body>
    <div class="parent">
        <div class="middle column">middle</div>
        <div class="left column">left</div>
        <div class="right column">right</div>
    </div>
</body>
```

### 双飞翼布局
中间盒子外面需要再包一层盒子，将外面这层盒子设置 margin 为左右两栏的宽度，三个盒子设置浮动，中间盒子宽度 100%，两边盒子宽度设置好之后，左边盒子设置 margin-left 为 100%，右边盒子设置 margin-left 为 -自身宽度，最后父盒子清除浮动。

好处是稳定，代码简洁，缺点是多加了 DOM 节点。

```html
<style>
    .container {
        overflow: hidden;
    }
    .content, .left, .right {
        float: left;
        height: 200px;
    }
    .content {
        width: 100%;
    }
    .center {
        margin: 0 200px;
        background-color: pink;
        height: 200px;
    }
    .left {
        width: 200px;
        margin-left: -100%;
        background-color: #eee;
    }
    .right {
        width: 200px;
        margin-left: -200px;
        background-color: #eee;
    }
</style>

<body>
    <div class="container">
        <div class="content">
            <div class="center">center</div>
        </div>
        <div class="left">left</div>
        <div class="right">right</div>
    </div>
</body>
```

## 3.11 link 和 @import 的区别
- link 属于 XHTML 标签，除了加载 CSS 外，还能用于定义 RSS(是一种描述和同步网站内容的格式，是使用最广泛的 XML 应用), 定义 rel 连接属性等作用
- 而 @import 是 CSS 提供的，只能用于加载 CSS
- 页面被加载的时，link 会同时被加载，而 @import 引用的 CSS 会等到页面被加载完再加载
- import 是 CSS2.1 提出的，只在 IE5 以上才能被识别，而 link 是 XHTML 标签，无兼容问题
总之，link 要优于 @import。

## 3.12 雪碧图
将一个页面设计到的图片包含到一张图片里去，然后通过定位的方法选取合适的图片。

这样做的好处是：
- 可以一次请求完所有图片，不用发很多请求，加快了速度，提升了页面性能
- 减少图片的字节，放在一起比单独的图片字节数更小
- 更换风格方便，只需要更换这张雪碧图所有的图标风格都可以改变

缺点是：
- 切图很麻烦，还可能造成背景断裂
- 开发和维护都比较麻烦，一个图标元素的位置移动可能导致所有图标位置都要移动

## 3.13 伪类和伪元素
根本不同在于是否创建了新的 DOM 元素
- 伪类操作文档中已有的元素，如 `:hover`、`:link`、`:focus`
- 伪元素创建新的元素进行操作，如 `::first-line`、`:before`、`:after`

## 3.14 [flex布局](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

容器的属性
- flex: wrap 决定如何换行，nowrap | wrap | wrap-reverse
- flex: direction 决定主轴的方向，row | row-reverse | column | column-reverse
- justify-content 决定主轴上的对齐方式
- align-items 决定交叉轴上的对齐方式

项目的属性
- grow：存在剩余空间时的放大倍数
- shrink：空间不足时的缩小比例
- basis：分配多余空间之前，主轴占据的空间
- flex: 1 包含了三个元素：grow, shrink, basis，默认值为 0 1 auto
    ```css
    .item {
        flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
    }
    ```

## 3.15 rem 和 em
- rem 是根据根节点的字体大小比例调整
- em 是根据父节点的字体大小进行比例调整

rem 经常被用于移动端响应式的布局，原理是通过 媒体查询 或 flexible.js，在屏幕尺寸发生变化时，改变 html 元素的字体大小。也可以和 vw、vh 配合使用，用 vw 设置 html 根元素字体大小，当窗口大小改变时实现自适应改变字体大小。

## 3.16 清除浮动
- 结尾插入空标签或 br，添加 `clear: both`
- 父级盒子定义 `overflow: hidden`
- 父级盒子 `display: table`
- after 伪元素清除浮动
    ```css
    .clearfix:after {
        content: "";
        display: block;
        height: 0;
        clear: both;
        visibility: hidden;
    }

    /* IE6、7 专有 */
    .clearfix {
        *zoom: 1;
    }
    ```
- before 和 after 双伪元素清除浮动
    ```css
    .clearfix:before,
    .clearfix:after {
        content: "";
        display: table;
    }

    .clearfix:after {
        clear: both;
    }

    .clearfix {
        *zoom: 1;
    }
    ```

## 3.17 元素消失
- visibility: hidden
- opacity: 0
- display: none

前两个元素存在但不显示，后面这个元素不渲染，不保留位置

## 3.18 position
属性值有 static | relative | fixed | absolute | sticky | inherit

static 和 relative 不脱离文档流，relative 用于定位偏移的位置也作为空间保留

fixed 和 absolute 脱离文档流，absolute 用于定位父元素不受影响，fixed 是相对于浏览器窗口进行定位

# 4. Vue
## 4.1 MVC 和 MVVM
### MVC
在 Controller 中响应用户对 View 的事件，Controller 调用 Model 的接口对数据进行操作，一旦 Model 发生变化 Model 就通知相关视图进行更新

- Model：数据和数据的处理方法
- View：视图，负责对数据的展示
- Controller：定义界面对用户输入的响应方式，连接模型和视图，用于控制应用程序的流程，处理用户的行为和数据上的改变

### MVVM
mvvm 把 view 和 model 的同步逻辑自动化，只需要告诉 view 的显示内容与 model 哪一部分对应即可。采用双向绑定。

- Model：数据和数据的处理方法
- View：视图，负责对数据的展示
- ViewModel：View 的变动，自动反映在 ViewModel

## 4.2 生命周期
Vue 有完整的生命周期，创建 -> 挂载 -> 更新 -> 销毁
- beforeCreated  
    开始创建数据，只有默认的生命周期钩子和默认事件，data 和 methods 还没有初始化
- Created  
    data 有了，可以访问数据和方法，但没有挂载到 DOM，可以操作数据
- beforeMount  
    挂载之前，编译好了，但还没有挂载到页面上
- Mounted  
    到此创建完成了，可以进行 DOM 操作了
- beforeUpdate  
    data 中数据已经更新了，但还没有更新到页面上，页面上还是旧的数据
- Updated  
    页面上的数据被更新了，页面和 data 中数据一致了
- beforeDestroy  
    销毁之前，适合销毁定时器等
- Destroyed  
    所有的数据、方法、过滤器、指令都不可用了，组件已经被销毁了

## 4.3 双向绑定
### Object.defineProperty
Object.defineProperty 进行数据劫持，遍历所有的属性给它们增加 getter 和 setter，组件的 data 发生变化时，将变化发布给订阅者，订阅者收到消息后进行处理

缺点：
- 无法监听数组通过下标改变对应数据
- 无法对新增加或删除的属性进行监听，需要使用 Vue.set()
- 一次性递归到底开销很大

### Proxy
使用 ES6 提供的 Proxy 进行数据拦截，劫持整个对象，然后返回一个新对象

解决无法监听新增属性或删除属性的响应式问题、解决无法监听数组长度和 index 变化问题。

## 4.4 Vue3新特性
- proxy 代替 Object.defineProperty
- 优化 diff 算法
- 生命周期不同了
- Vue3 使用组合式 API
- 支持 TS

## 4.5 路由
### history
美观但兼容性略差，部署上线可能会出现 404 问题。

解决方法：使用 node 的 connect-history 中间件

### hash
哈希模式 # 后面的路径不发送给服务器，不太美观，有可能被 APP 标记为不合法，但兼容性好。

## 4.6 路由守卫
作用：保护路由的安全，权限问题

### 全局路由守卫
分为`beforeEach、beforeResolve、afterEach`

使用：全局前置路由守卫`beforeEach` 进行权限校验

### 独享路由守卫
只有前置守卫`beforeEnter`，是在路由配置页面单独给路由配置的一个守卫

使用：不同的路由重用守卫，`beforeEnter` 守卫只在进入路由时触发

### 组件内路由守卫
分为进入守卫和离开守卫`beforeRouteEnter`、`beforeRouteLeave`、`beforeRouteUpdate`

使用：
- `beforeRouteLeave`：离开页面时弹出提示窗口，清除当前组件中的定时器等；关闭页面时, 将公用信息保存到 session 或 Vuex 中；当前页面中有未关闭的窗口或未保存的内容阻止页面跳转
- `beforeRouteUpdate`：动态路由跳转

## 4.7 Vuex
用于集中式管理数据，一般用于大中型应用

- state：记录数据状态
- getters：获取数据，可以根据 state 中的数据进行过滤派生出一些新的数据
- action：进行异步操作
- mutations：唯一可以改变 state 中数据的方法
- module：如果数据较多可以分模块

## 4.8 nextTick
Vue在更新数据时是异步执行的，data 中数据更新了但页面还没来得及更新，所以如果立刻获取 DOM，获取到的是还没有更新的 DOM，nextTick 就是等数据更新之后再进行 DOM 操作。本质是返回一个 Promise

## 4.9 Diff算法
Diff 算法用来找出虚拟 DOM 中被改变的部分，然后针对原生 DOM 进行渲染，不用改变所有节点重新渲染整个页面

比较方式
- 同层比较：两个树的完全的 diff 算法是一个时间复杂度为 O(n^3) 的问题。但是在前端当中，你很少会跨越层级地移动 DOM 元素。所以 Virtual DOM 只会对同一个层级的元素进行对比，这样算法复杂度就可以达到 O(n)。
- 深度优先：在实际的代码中，会对新旧两棵树进行一个深度优先的遍历，这样每个节点都会有一个唯一的标记。在深度优先遍历的时候，每遍历到一个节点就把该节点和新的的树进行对比。如果有差异的话就记录到一个对象里面。

## 4.10 组件间通信
- 父传子：props
- 子传父：$emit, $on
- 任意组件间
    - vuex
    - eventbus：使用方法是创建一个新的Vue实例，需要通信的组件都引入该Vue实例，传递数据的组件使用 ` event.$emit('名称',参数)` 发送数据，接收数据的组件使用 `event.$on('名称',方法)` 接收数据
    - 订阅发布模式

## 4.11 keep-alive
用 `<keep-alive>` 标签对需要缓存的组件进行包裹，保存内存中组件的状态，进行缓存，防止重新加载 DOM，减少加载时间，提高性能。

多了两个生命周期：activated、deactivated，分别在进入和退出时触发。

属性：
- include：缓存包含哪些组件
- exclude：缓存不包含哪些组件
- max：最多保存的组件数

## 4.12 v-if 和 v-show
- v-if 一开始不渲染，不在 DOM 树中，节点要显示出来才开始渲染，渲染一次消耗很大
- v-show 一开始渲染好了只是不显示，适用于隐藏显示操作频繁的情况

## 4.13 为什么 data 不能是一个对象而是一个匿名函数？
JS 中对象是引用类型的数据，而在 Vue 的组件中，我们更多考虑的是组件的复用，所以每个 data 都要有自己独立的存储空间，这样组件之间才不会相互干扰。所以组件中要写成函数的形式，数据以函数的返回值定义，这样每次复用组件的时候就会返回新的 data。

## 4.14 为什么 v-for 和 v-if 不要一起使用？
v-for 的优先级比 v-if 更高，如果放在一起使用，就算只需要渲染一小部分 v-if，每次 v-for 的计算也都会执行一遍所有的 v-if，造成不必要的计算，影响性能。

# 5. Webpack

## 入口和出口
配置入口和出口文件
```js
var path =require('path')

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'main.js'
    },
};
```

多个入口文件的配置
- 多 -> 一
    ```js
    var path =require('path')
    module.exports = {
        entry: ['./src/a.js', './src/b.js'],
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'main.js'
        },
    };
    ```

- 多 -> 多
    ```js
    var path =require('path')
    module.exports = {
        entry: {
            a: './src/a.js', 
            b: './src/b.js'
        },
        output: {
            path: path.resolve(__dirname, 'dist'),
        },
    };
    ```

## loader
loader 让 webpack 可以处理除了 js 文件和 JSON 文件以外的其他类型的文件，将它们转换为有效模块

loader 有两个属性：
1. test 识别出需要进行转换的文件
2. use 定义在进行转换时用哪个 loader

```js
var path =require('path')

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'main.js'
    },
    module: {
        rules: [{
            test: /\.txt&/,
            use: ['style-loader', 'css-loader']
        }]
    }
};
```

## 插件 plugin
html-webpack-plugin

```js
var HTMLWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    plugins: [
        new HTMLWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html'
        })
    ]
}
```

# 6. 数据结构
## 6.1 [常用的数据结构类型有哪些](https://zh.m.wikipedia.org/zh-hans/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
数组、栈、堆、队列、链表、散列表、树、图

## 6.2 二分查找
```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var search = function(nums, target) {
  let left = 0;
  let right = nums.length-1;
  
  while(left <= right) {
    let mid = left+((right-left) >> 1);
    if (nums[mid] === target) return mid;
    else if (nums[mid] > target) {
      right = mid-1;
    } else if (nums[mid] < target) {
      left = mid+1;
    }
  }
  return -1;
};
```

## 6.3 排序 
【待总结】

### 冒泡排序
从第一个元素开始，把当前元素和下一个索引元素进行比较。如果当前元素大，那么就交换位置，重复操作直到比较到最后一个元素，那么此时最后一个元素就是该数组中最大的数。下一轮重复以上操作，但是此时最后一个元素已经是最大数了，所以不需要再比较最后一个元素，只需要比较到 `length - 1` 的位置。
```js
function bubble(arr) {
    for(let i = 0; i < arr.length - 1; i++) {
        for(let j = 0; j < arr.length; j++) {
            if(arr[j] > arr[j+1]) {
                let tmp = arr[j+1];
                arr[j+1] = arr[j];
                arr[j] = tmp;
            } 
        }
    }
    return arr;
}
```
冒泡每次只关注一个元素，两次循环，平均时间复杂度 \\( O(n^2) \\)，空间复杂度 \\( O(1) \\)，该算法是稳定的。

### 插入排序
插入排序的原理如下。第一个元素默认是已排序元素，取出下一个元素和当前元素比较，如果当前元素大就交换位置。那么此时第一个元素就是当前的最小数，所以下次取出操作从第三个元素开始，向前对比，重复之前的操作。
```js
function insertion(arr) {
    for(let i = 1; i < arr.length; i++) {
        for(let j = i-1; j >= 0; j--) {
            if(arr[j] > arr[j+1]) {
                let tmp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = tmp;
            }
        }
    }
    return arr;
}
```
直接插入排序是稳定的，平均时间复杂度 \\( O(n^2) \\)，空间复杂度 \\( O(1) \\)

### 选择排序
选择排序的原理如下。遍历数组，设置最小值的索引为 0，如果取出的值比当前最小值小，就替换最小值索引，遍历完成后，将第一个元素和最小值索引上的值交换。如上操作后，第一个元素就是数组中的最小值，下次遍历就可以从索引 1 开始重复上述操作。
```js
function selection(arr) {
    for(let i = 0; i < arr.length; i++) {
        let minIndex = i;
        for(let j = i+1; j < arr.length; j++) {
            minIndex = arr[j] < arr[minIndex] ? j : minIndex;
        }
        let tmp = arr[minIndex];
        arr[minIndex] = arr[j];
        arr[j] = tmp;
    }
    return arr;
}
```
直接选择排序不稳定，平均时间复杂度 \\( O(n^2) \\)，空间复杂度 \\( O(1) \\)

### 归并排序
归并排序的原理如下。递归的将数组两两分开直到最多包含两个元素，然后将数组排序合并，最终合并为排序好的数组。假设我有一组数组 `[3, 1, 2, 8, 9, 7, 6]`，中间数索引是 3，先排序数组 `[3, 1, 2, 8]` 。在这个左边数组上，继续拆分直到变成数组包含两个元素（如果数组长度是奇数的话，会有一个拆分数组只包含一个元素）。然后排序数组 `[3, 1]` 和 `[2, 8]` ，然后再排序数组 `[1, 3, 2, 8]` ，这样左边数组就排序完成，然后按照以上思路排序右边数组，最后将数组 `[1, 2, 3, 8]` 和 `[6, 7, 9]` 排序。
```js
function mergeSort(arr) {

}
```

### 快排
随机选取一个数组中的值作为基准值，从左至右取值与基准值对比大小。比基准值小的放数组左边，大的放右边，对比完成后将基准值和第一个比基准值大的值交换位置。然后将数组以基准值的位置分为两部分，继续递归以上操作。
```js
function sort(arr) {
    
}
```
### 几种排序方法复杂度和稳定性比较
|排序方法|平均时间复杂度|空间复杂度|稳定性|
|----|----|----|----|
|冒泡排序|\\( O(n^2) \\)|\\( O(1) \\)|稳定|
|直接插入排序|\\( O(n^2) \\)|\\( O(1) \\)|稳定|
|直接选择排序|\\( O(n^2) \\)|\\( O(1) \\)|不稳定|
|快速排序|\\( O(nlogn) \\)|\\( O(nlogn) \\)|不稳定|
|归并排序|\\( O(nlogn) \\)|\\( O(n) \\)|稳定|

## 6.4 二叉树