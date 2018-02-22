---
layout:     post
title:      "订阅(subscribe)/发布(publish)模式"
subtitle:   ""
date:       2018-02-08 12:00:00
author:     "Hooke"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术
---

# 订阅(subscribe)/发布(publish)模式

 sub 会监听消息当 pub 发布的时候进行相应的逻辑操作, 他们中间有调度中心的存在.

 ``` javascript
 var scope = (function () {
  // 消息列表
  var events = {}
  return {
    // 消息订阅
    on (name, handler) {
      // 记录当前消息事件的索引
      var index;
      // 如果该消息已经存在, 则将处理函数放到该消息的事件队列中
      if (events[name]) {
        index = events[name].push(handler) - 1
      } else {
        // 如果该消息已经不存在, 则创建该消息的事件队列, 并将处理函数放到该消息的事件队列中
        events[name] = [handler]
        index = 0
      }
      // 返回当前消息处理事件的移除函数
      return function () {
        events[name].splice(index, 1)
      }
    },
    // 消息关闭
    off (name) {
      if (!events[name]) return;
      // 如果该消息存在, 则将该消息删除
      delete events[name];
    },
    // 消息发布
    emit (name, msg) {
      // 如果该消息不存在, 不处理
      if (!events[name]) return;
      // 该消息存在, 将该消息事件队列中的事件都处理一遍
      events[name].forEach((v, i) => {
        v(msg)
      })
    }
  }
 })()

 var remove = scope.on('refreshList', function (msg) {

 })

 // 点击刷新
 var refresh = function () {
  scope.emit('refreshList')
 }

 // 添加数据
 var addItem = function () {
  scope.emit('refreshList')
 }

 // 删除数据
 var removeItem = function () {
  scope.emit('refreshList')
 }

 var destroy = function () {
  // 移除 refreshList 中对应的事件
  remove()
  // 移除 refreshList
  scope.off('refreshList')
 }
 ```
