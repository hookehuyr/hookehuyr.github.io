---
layout:     post
title:      "ES6 Promise 和 Async/await"
subtitle:   ""
date:       2018-03-15 12:00:00
author:     "Hooke"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术
---

# ES6 Promise 和 Async/await
Javascript语言的执行环境是"单线程"（single thread）。

所谓"单线程"，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。

这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段Javascript代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

为了解决这个问题，Javascript语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。

## 回调

回调是异步编程最基本的方法。

假定有两个函数f1和f2，后者等待前者的执行结果。

```javascript
f1();
f2();
```

如果f1是一个很耗时的任务，可以考虑改写f1，把f2写成f1的回调函数。

```javascript
function f1(callback){
　　setTimeout(function () {
　　　　// f1的任务代码
　　　　callback();
　　}, 1000);
}
```

执行代码就变成下面这样

```javascript
f1(f2);
```

采用这种方式，我们把同步操作变成了异步操作，f1不会堵塞程序运行，相当于先执行程序的主要逻辑，将耗时的操作推迟执行。
回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合，流程会很混乱，而且每个任务只能指定一个回调函数。

## Promise
`Promises`对象是`CommonJS`工作组提出的一种规范，目的是为异步编程提供统一接口。

简单说，它的思想是， 每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。 `Promises`的出现大大改善了异步变成的困境，避免出现回调地狱，嵌套层级得到改善。

### 基本Api
- Promise.resolve()
- Promise.reject()
- Promise.prototype.then()
- Promise.prototype.catch()
- Promise.all()  // 所有的完成
- Promise.race() // 竞速，完成一个即可

### 模拟两个异步请求
```javascript
// 1请求
function getData1 () {
  return new Promise(function (resolve, reject) {
    setTimeout(() => {
      console.log('1执行了')
      resolve('请求到模拟数据1111拉')
    }, 2000)
  })
}
// 2请求
function getData2 (params) {
  return new Promise(function (resolve, reject) {
    setTimeout(() => {
      console.log('2执行了')
      resolve('请求到模拟数据22222拉！params：' + params)
    }, 1500)
  })
}

```

### promise 实现异步回调 异步列队
1请求完成后，把1的响应参数传入2，在发2请求
```javascript
function promiseDemo () {
  getData1()
    .then(res => {
      return getData2(res)
    })
    .then(res => {
      console.log(res)
    })
}
promiseDemo()
// 1执行了
// 2执行了
// 请求到模拟数据22222拉！params：请求到模拟数据1111拉   用时 3500 ms
```

### promise.all() 实现异步回调 并发 所有的完成
1请求、2请求同时发,两条响应都收到后在执行
```javascript
function promiseDemo () {
  Promise.all([getData1(), getData2()]).then(function (res) {
    console.log(res)
  })
}
// 2执行了
// 1执行了
// ["请求到模拟数据1111拉", "请求到模拟数据22222拉！params：undefined"]   用时 2000 ms
```
### promise.race() 实现异步回调 并发 竞速
1请求、2请求同时发，其中一条收到请求就执行
```javascript
function promiseDemo () {
  Promise.race([getData1(), getData2()]).then(function (res) {
    console.log(res)
  })
}
// 2执行了
// 请求到模拟数据22222拉！params：undefined    用时 1500 ms
// 1执行了   
```

## Async/await
Async/await 是Javascript编写异步程序的新方法。以往的异步方法无外乎回调函数和Promise。但是Async/await建立于Promise之上。

```javascript
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```
上面代码指定50毫秒以后，输出hello world。 进一步说，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖

### async 实现异步回调 异步列队
1请求完成后，把1的响应参数传入2，在发2请求

上文中的promise 实现方法是通过then的链式调用，但是采用async会更加简洁明了

```javascript
async function asyncDemo () {
  const r1 = await getData1()
  const r2 = await getData2(r1)
  console.log(r2)
}
// 1执行了
// 2执行了
// 请求到模拟数据22222拉！params：请求到模拟数据1111拉   用时 3500 ms
```
用同步的书写方式实现了异步的代码。等待getData1的异步函数执行完了后发返回值赋值给r1，传入r2,在执行r2

###
async 异步回调 并发
```javascript
async function asyncDemo2 () {
  const arr = [getData1, getData2]
  const textPromises = arr.map(async function (doc) {
    const response = await doc()
    return response
  })
  // 按次序输出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
// 2执行了            (因为2是 1500ms后执行) 所以2先执行
// 1执行了
// 请求到模拟数据1拉  (for .. of )规定了输出的顺序
// 请求到模拟数据22222拉！params：undefined
```

虽然map方法的参数是async函数，但它是并发执行的，因为只有async函数内部是继发执行，外部不受影响。后面的for..of循环内部使用了await，因此实现了按顺序输出
