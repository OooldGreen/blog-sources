---
title: "期末知识点复习"
date: 2022-10-30T10:35:16+08:00
draft: false

keywords: ["interview"]
description: ""
tags: ["interview"]
categories: ["Frontend"]
author: ""

hidemeta: false
comments: true
# canonicalURL: "https://canonical.url/to/page"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: false
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
- HTTPS更安全，更利于 SEO
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
#### Cache-Control：
- 设置 max-age，设定时间内可以直接从本地缓存中读取文件并加载
- public/private，设置缓存能否在用户间共享
- no-cache， 不进行缓存

#### Expires：
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
- 内联图片：利用js检查标签是否在视窗中，如果在的src设置为目标图片的url
- intersection observer：要考虑兼容性
- css图像：利用background-image，当ja检测到元素处于视窗中，加一个class类名引用外部图片资源

#### 路由懒加载
动态导入代替静态导入

```js
// 将
import UserDetails from './views/UserDetails'
// 替换成
const UserDetails = () => import('./views/UserDetails')
```

使用webpack（babel）分块导入，只需要使用命名 chunk，魔法注释提供chunk name 

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
3. 有利于SEO，提高搜索效率

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

改变原数组：push、pop、shify、unshift、reverse、sort、splice

不改变原数组：concat、join、filter、slice、reduce

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
- for in通用的对象遍历，遍历对象的key，也包括原型链上的属性的遍历，是以任意顺序迭代对象的可枚举属性
- for of遍历的是值，遍历可迭代属性，不能直接用来遍历对象

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

#### 方法：
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
- 异步promise
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
- 微任务 (microtask)：promise的回调、nextTick、mutationObserver

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
- apply 接收参数是一个数组，call 和 bind 可以接收多个参数

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

## 3.2 HTML 语意化

## 3.3 HTML 中 title 属性和 alt 属性的区别

## 3.4 回流和重绘

## 3.5 BFC

## 3.6 CSS3新特性

## 3.7 盒子模型

## 3.8 垂直居中

## 3.9 画一个三角形

## 3.10 三栏布局

## 3.11 link 和 @import 的区别

## 3.12 雪碧图

## 3.13 伪类和伪元素

## 3.14 flex布局

## 3.15 rem 和 em

## 3.16 清除浮动

## 3.17 元素消失

## 3.18 position

# 4. Vue
## 4.1 MVC 和 MVVM

## 4.2 生命周期

## 4.3 双向绑定

## 4.4 Vue3新特性

## 4.5 路由

## 4.6 路由守卫

## 4.7 Vuex

## 4.8 nextTick

## 4.9 Diff算法

## 4.10 组件间通信