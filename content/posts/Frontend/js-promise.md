---
title: "Promise"
date: 2022-11-06T15:35:31+08:00
lastmod: 2022-11-06
draft: false
keywords: ["frontend", "promise", "javascript", "es6"]
description: ""
tags: ["JS", "promise"]
categories: ["Frontend"]
author: ""

---

汇总`Promise`对象的由来、用法、方法和`async/await`

从是什么，怎么用开始讲起啦，然后讲讲具体有哪些方法和应用场景，最后再浅提一下`async/await`

<!--more-->

# 为什么要有Promise对象

一句话：解决回调地狱(callback hell)

如果你想进行一个异步操作，比方说，有先后顺序的打印两个结果，你会怎么写？

也许你可以使用回调函数（callback），因为回调函数必须依赖另一个函数执行，我们可以借此确保打印的先后顺序。定时器就是一个常见的回调函数。
```js
// 1s后输出callback
setTimeout(function () { 
    console.log('callback')
}, 1000);
```
当要嵌套很多回调函数时，每个嵌套都用一层括号包裹，最后的结果就是嵌套了很多层括号，代码变得非常复杂，令人迷惑...... 摘抄[JavaScript高级程序设计（第4版）](https://book.douban.com/subject/35175321/)上的一段代码体会一下，这段代码表现的是异步返回值依赖了另一个异步返回值时的情况——

```js
function double(value, success, failure) {

setTimeout(() => { 
  try { 
    if (typeof value !== 'number') { 
      throw 'Must provide number as first argument'; 
    } 
    success(2 * value); 
  } catch (e) { 
    failure(e); 
  } }, 1000);
}

const successCallback = (x) => { 
  double(x, (y) => console.log(`Success: ${y}`)); 
}; 
const failureCallback = (e) => console.log(`Failure: ${e}`);

double(3, successCallback, failureCallback);

// Success: 12（大约 1000 毫秒之后）
```

这个时候如果可以推出一种方法，可以方便又直观的表示出异步操作的先后顺序，又不用一层一层的写回调函数就好了......

这个方法就是 ES6 推出的 Promise

# Promise对象是什么
Promise是一个异步操作，通俗来说就是Promise里面的程序运行完根据返回值再进行下一步操作，这样不用一直在函数体里一直嵌套回调。

一共有三种状态：`PENDING`（待定）、`FULFILLED/RESOLVED`（成功）、`REJECTED`（失败）。这是一个“承诺”，从一开始的`PENDING`开始，状态一旦改变便不会再变，成败只在一线之间（bushi。所以最后结果就是返回`RESOLVED`或者`REJECTED`，一般来说返回`RESOLVED`进行下一步，返回则`REJECTED`抛出错误。

# Promise对象怎么用

用`new`实例化，接住`resolve`或`reject`
```js
const p1 = new Promise((resolve, reject) => {
  if (/* 操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
})
```

## Promise 实例方法

### Promise.prototype.then()
then 是 Promise 的实例方法，在它的实例对象上，作用是作为 Promise 状态改变后的回调函数使用。

then 接收最多两个参数，一个是 `resolved` 后的操作，一个是 `rejected` 后的操作。这两个参数都是可选的，如果提供的话就会分别在进入两个状态后返回一个新的 Promise，因此可以使用链式调用，也就是在 then 方法后再调用 then方法，写很多个 then 方法也没问题。

采用链式调用就会按先后顺序执行，因此可以规定一组回调函数按照次序进行调用，也就是第一个回调执行完之后，返回新的 Promise 作为下一个回调的参数传入第二个回调函数，以此类推。

```js
const p1 = new Promise((resolve, reject) => {
  resolve('Success!');
});

p1.then((value) => {
  console.log(value); // "Success!"
});
```

### Promise.prototype.catch()
用于指定发生错误时的回调

```js
const p1 = new Promise((resolve, reject) => {
  throw new Error('error!');
}).catch(e => {
  console.log(e);
});
```


## Promise 静态方法

### Promise.all()
用于一组 Promise 实例，当所有实例状态都为 `fulfilled` 才会返回成功，否则会抛出状态为 `rejected` 的实例返回的错误

```js
const p = Promise.all([p1, p2, p3, ...])
```

### Promise.any()
用于一组 Promise 实例，当其中一个实例状态转变为 `fulfilled`，该实例状态就变为 `fulfilled`；当所有实例的状态都为 `reject`，实例才会变为 `reject` 状态。

```js
const p = Promise.any([p1, p2, p3, ...])
```

### Promise.race()
用于一组 Promise 实例，实例的状态根据第一个改变状态的 Promise 改变

```js
const p = Promise.race([p1, p2, p3, ...])
```

### Promise.resolve()
将现有对象转变为 Promise 实例对象。

如果参数是一个 Promise，返回执行后新的 Promise 对象；  
如果参数是一个带有 `then` 方法的对象，则将这个对象转为 Promise 后立即执行 `then` 方法；  
如果参数根本就不是对象，则返回一个新的 Promise 对象，状态为 `resolved`；  
如果参数不带任何参数，则直接返回一个状态为 `resolved` 的 Promise 对象。

### Promise.reject()
用于返回失败后的实例对象，状态为 `rejected` ，并给定后续的处理方法。

# 手写实现 Promise
## 实现 Promise
```js
// 定义三种状态
const PENDING = 'pending';
const REJECTED = 'rejected';
const RESOLVED = 'resolved';

class MyPromise {
  constructor(executor) {
    this.status = PENDING;

    // 存储成功和失败的结果
    this.value = undefined;

    // 成功和失败的回调函数
    this.resolvedCallbacks = [];
    this.rejectedCallbackes = [];

    const resolve = (value) => {
      if (this.status === PENDING) {
        this.status = RESOLVED;
        this.value = value;
        this.resolvedCallbacks.forEach(cb => cb(value))
      }
    }

    const reject = (value)=> {
      if(this.status = PENDING) {
        this.status = REJECTED;
        this.reason = value;
        this.rejectedCallbaecks.forEach(cb => cb(value))
      }
    }

    try {
      executor(resolve, reject);
    } catch(e) {
      reject(e)
    }
  }
  // 实现 then 函数
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) {
      if(this.status === RESOLVED) {
        onFulfilled(this.value);
      }
      if(this.status === REJECTED) {
        onRejected(this.value);
      }
      if(this.status === PENDING) {
        this.resolvedCallbacks.push(onFullfilled);
        this.rejectedCallbacks.push(onRefected);
      }
    });
  }
}
```

## 实现 Promise.all()

```js
class MyPromise {
  static all(promises) {
    let results = [];
    let resolvedCounter = 0;

    return new Promise((resolve, reject) => {
      promises.forEach((promise, index) => {
        promise.then(value => {
          results[index] = value;
          resolvedCounter++;

          if(resolvedCounter === promises.length) {
            return resolve(results);
          }
        }).catch(e => {
          reject(e);
        });
      });
    });
  }
}
```

# async/await 又是啥？
async/await 其实是 Generator 的语法糖，可以让我们以更简洁的方式写出异步行为，解决了“链式调用地狱”，可能是终极解决方法。

async 关键字用于声明异步函数，也就是说它是一种标识符，写在函数前面表明函数里面有需要异步执行的操作。

await 关键字用于暂停异步函数代码的执行，等待 await 右边的 Promise 被解决或被拒绝后再异步回复函数进程（即使 await 右边只是一个普通的值，也会被隐式地被包装成一个 Promise 并异步执行）。

在错误处理机制中，async/await 可以使用 try/catch 代码块，用 catch 捕捉到错误。

利用 async/await 实现 sleep()
```js
async function sleep(delay) {
  return await new Promise(resolve => setTimeout(resolve, delay));
}

// 使用
async function foo() {
  await sleep(1500);
  console.log('Hello World :)');
}
foo(); 
// 暂停 1500ms 后输出 'Hello World :)' 
```

## 参考
- [JavaScript高级程序设计（第4版）](https://book.douban.com/subject/35175321/)
- [阮一峰老师的ES6入门](https://es6.ruanyifeng.com/#docs/promise)
- [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
