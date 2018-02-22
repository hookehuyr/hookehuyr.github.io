---
layout:     post
title:      "事件模型"
subtitle:   ""
date:       2018-02-23 12:00:00
author:     "Hooke"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术
---

# 事件模型
事件是一种异步编程的实现方式，本质上是程序各个组成部分之间的通信。

## EventTarget接口
1. addEventListener: 绑定事件监听函数
2. removeEventListener: 移除事件监听函数
3. dispatchEvent: 触发事件

### addEventListener()
默认格式为: `target.addEventListener(type, listener[, useCapture])`
> type 事件名称, 大小写敏感
> listener 监听事件
> useCapture 默认监听事件在冒泡阶段触发, 但是为了兼容老浏览器建议写上

### removeEventListener()
移除事件监听, 注意必须和 `addEventListener` 设置时一致.

### dispatchEvent()
1. 如果监听函数调用过 `Event.preventDefault()`, 它的返回值为 `false` 否则为 `true`
2. 它必选传一个有效事件对象即 `target.dispatch(event)` 否则会报错

```javascript
// 1
var canceled = !ele.dispatch(event)
if (canceled) {
  console.log('监听事件调用过取消默认事件')
} else {
  console.log('监听事件没有调用过')
}
// 2
ele.addEventListener('click', fn, false)
var event = new Event('click')
ele.dispatch(event)
```

## 监听函数
即事件触发时执行的函数, 属于事件驱动编程
DOM提供三种方式绑定监听函数

1. HTML标签的on-属性
- 它绑定的是一个执行函数, 如 `<div onclick="dosomething()">` 不能是 `<div onclick="dosomething">`, 缺点很明显它违背了结构和逻辑相分离援助, 而且函数执行时可能会造成 `this` 指向的混乱. 只能冒泡阶段触发
- 和这种写法实质上是一致 `ele.setAtrribute('onclick', dosomething())`

2. Element节点的事件属性
- `window.onload = dosomething` 它是把函数指向指到了节点上, 但是它也只在冒泡阶段触发.
- 它的缺点是一个事件不能绑定多个函数, 绑定会把上一次的函数冲掉.

3. addEventListener方法
通过`Element`节点、`document`节点、`window`对象的`addEventListener`方法，也可以定义事件的监听函数.

## 事件传播

### 传播的三个阶段

### 事件的代理

### Event对象
  #### event.bubbles, event.eventPhase
  #### event.cancelable, event.defaultPrevented
  #### event.currentTarget, event.target
  currentTarget: 绑定的节点
  target: 触发的节点
  #### event.type，event.detail，event.timeStamp，event.isTrusted
  #### event.preventDefault()
  #### event.stopPropagation() 阻止事件进一步传播
  #### event.stopImmediatePropagation() 阻止同一个事件的其他监听函数被调用

### 自定义事件和事件模拟
  ```javascript
  // 新建事件实例
  var event = new Event('build')
  // 添加监听函数
  ele.addEventListener('build', function (e) {}, false)
  // 触发事件
  ele.dispatchEvent(event)
  ```
  #### CustomEvent() 初始化事件名称并传入指定数据
  ```javascript
  var evt = new CustomEvent('myevent', {
    detail: {
      foo: 'bar'
    },
    bubbles: true,
    cancelable: false
  })
  ele.addEventListener('myevent', function (e) {
    console.log(e.detail.foo)
  })
  ele.dispatchEvent(evt)
  ```
  #### 事件模拟 模拟鼠标点击事件
  ```javascript
  function simulteClick () {
    var evt = new MouseEvent ('click', {
      bubbles: true,
      cancelable: true
    })
    var cb = document.getElementById('checkbox')
    cb.dispatchEvent(evt)
  }
  ```

  ### 自定义事件的老式写法

  事件类型 | 事件初始化方法
  --------- | --------------
  UIEvents | event.initUIEvent
  MouseEvents | event.initMouseEvent
  ~~MutationEvents~~ | ~~event.initMutationEvent~~
  HTMLEvents | event.initEvent
  Event |	event.initEvent
  CustomEvent |	event.initCustomEvent
  KeyboardEvent |	event.initKeyEvent

  ```javascript
  // 新建Event实例
  var event = document.createEvent('Event');
  // 事件的初始化
  event.initEvent('build', true, true);
  // 加上监听函数
  document.addEventListener('build', doSomething, false);
  // 触发事件
  document.dispatchEvent(event);
  ```

  ```javascript
  // 初始化自定义事件
  var event = document.createEvent('Event');
  event.initEvent('my-custom-event', true, true, {foo:'bar'});
  someElement.dispatchEvent(event);
  ```

  ### 事件模拟的老式写法
  `event.initMouseEvent()`

  ```javascript
  // 参数
  event.initMouseEvent(type, canBubble, cancelable, view,
    detail, screenX, screenY, clientX, clientY,
    ctrlKey, altKey, shiftKey, metaKey,
    button, relatedTarget
  );
  // 模拟鼠标事件
  var simulateDivClick = document.createEvent('MouseEvents');

  simulateDivClick.initMouseEvent('click',true,true,
    document.defaultView,0,0,0,0,0,false,
    false,false,0,null,null
  );

  divElement.dispatchEvent(simulateDivClick);
  ```

  `UIEvent.initUIEvent()`

  ```javascript
  // 参数
  event.initUIEvent(type, canBubble, cancelable, view, detail)
  // 初始化 UI 事件
  var e = document.createEvent("UIEvent");
  e.initUIEvent("click", true, true, window, 1);
  ```
