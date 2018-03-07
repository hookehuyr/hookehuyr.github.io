---
layout:     post
title:      "Webpack 插件API - Plugin API"
subtitle:   ""
date:       2018-03-07 12:00:00
author:     "Hooke"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术
    - webpack
---

# Webpack 插件API - Plugin API
webpack 中的很多对象继承了 `Tapable` 类，暴露出一个 `plugin` 方法。通过 `plugin` 方法，插件可以注入自定义的构建步骤。你会看到 `compiler.plugin` 和 `compilation.plugin` 被大量使用。本质上，每一个 plugin 方法调用时的回调函数的触发时机，都会被绑定到整个构建过程中特定步骤。

有两种类型的 plugin 接口……

## 基于时间的(Timing Based)

* sync（默认）：plugin 同步执行，返回它的输出
* async：plugin 异步执行，使用给定的 callback 来返回它的输出
* parallel：处理函数被并行地调用

## 返回值的(Return Value)

* not bailing（默认）：没有返回值
* bailing：处理函数被顺序调用，直到一个处理函数有返回值
* parallel bailing：处理函数被并行（异步）地调用。（按顺序）第一个被返回的值会被使用。
* waterfall：每个处理函数，将上一个处理函数的结果作为参数。

plugin 会在 webpack 启动时被安装。webpack 通过调用 plugin 的 `apply` 方法来安装它，传入一个指向 webpack `compiler` 对象的引用。你可以调用 `compiler.plugin` 来访问 asset 的编译(compilation)和它们各自的构建步骤。例如：

my-plugin.js


```javascript
function MyPlugin(options) {
  // 使用 options 配置你的 plugin
}

MyPlugin.prototype.apply = function(compiler) {
  compiler.plugin("compile", function(params) {
    console.log("编译器开始编译(compile)……");
  });

  compiler.plugin("compilation", function(compilation) {
    console.log("编译器开始一个新的编译过程(compilation)……");

    compilation.plugin("optimize", function() {
      console.log("编译过程(compilation)开始优化文件...");
    });
  });

  compiler.plugin("emit", function(compilation, callback) {
    console.log("编译过程(compilation)准备开始输出文件……");
    callback();
  });
};

module.exports = MyPlugin;
```
