---
layout:     post
title:      "JavaScript 中的 call apply bind"
subtitle:   ""
date:       2018-02-06 12:00:00
author:     "Hooke"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术
---

# JavaScript 中的 call apply bind

## 函数的三个角色

- 普通函数
- 类
- 普通对象

## call
### 基本使用
call 的执行过程: 首先找到 Function.prototype 上的 call 方法, 执行 call 方法, 执行中把 fn 中的 this 换成 obj, 最后 fn 执行.

```javascript
var obj = {
  name: 'xxx'
}
function fn () {
  console.log(this.name)
}
fn.call(obj)
```
### call方法原理
### call实例
```javascript
function f1 () { console.log(1) }
function f2 () { console.log(2) }

// 输出1
f1.call(f2) // 1

// 输出2
f1.call.call(f2) // 2
```
### call、apply、bind的区别
ES5中使用严格模式, `use strict`
```javascript
function fn () {
  console.log(this)
}
fn.call() // 普通模式为 window 严格模式为 undefined
fn.call(null) // 普通模式为 window 严格模式为 null
fn.call(undefined) // 普通模式为 window 严格模式为 undefined
```
call 与 apply的方法作用一模一样, 都是用来改变方法的 this 关键字, b 并且把方法执行. 唯一不同的是参数的使用.
```javascript
function fn(num1, num2) {
  console.log(num1 + num2);
  console.log(this);
}
fn.call(obj , 100 , 200);
fn.apply(obj , [100, 200]);
```
`bind`的区别在于它会把 this 改变为我们想要的结果但是不会立即执行
```javascript
var temp = fn.bind(obj, 1, 2)
temp()
```
### call, apply 的应用
#### 求数组的最大值和最小值
```javascript
var ary = [23, 34, 24, 12, 35, 36, 14, 25];
var max = Math.max.apply(null, ary);
var min = Math.min.apply(null, ary);
console.log(min, max);
```
#### 求平均数
```javascript
function avgFn() {
    Array.prototype.sort.call(arguments , function (a, b) {
        return a - b;
    });

    [].shift.call(arguments);
    [].pop.call(arguments);

    return (eval([].join.call(arguments, '+')) / arguments.length).toFixed(2);
}
var res = avgFn(9.8, 9.7, 10, 9.9, 9.0, 9.8, 3.0);
console.log(res);
```
`call` 配合 `argument` 来使用, 并且调用数组方法居多, `argument` 并不是一个真正的数组.
`Array.prototype.slice.call(argument)` 或者 `[].slice.call(argument)` 都可以克隆数组.
在把类数组转换成数组可以用上面的方法, 但是 `IE6 ~ IE8` 就不行了. 可以使用下面的方法
```javascript
function likeToArray (likeArr) {
  var arr = []
  try {
    arr = [].slice.call(likeArr)
  }
  catch (e) {
    for (var i=0; i<likeArr.length; i++) {
      arr[arr.length] = likeArr[i]
    }
  }
  return arr
}
```
