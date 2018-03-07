---
layout:     post
title:      "Webpack 插件API - Compilation"
subtitle:   ""
date:       2018-03-07 12:00:00
author:     "Hooke"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术 webpack
---

# Webpack 插件API - Compilation
Compilation 实例继承于 compiler。例如，compiler.compilation 是对所有 require 图(graph)中对象的字面上的编译。这个对象可以访问所有的模块和它们的依赖（大部分是循环依赖）。在编译阶段，模块被加载，封闭，优化，分块，哈希和重建等等。这将是编译中任何操作主要的生命周期。

```javascript
compiler.plugin("compilation", function(compilation) {
    // 主要的编译实例
    // 随后所有的方法都从 compilation.plugin 上得来
});
```

## normal-module-loader
普通模块 loader，真实地一个一个加载模块图(graph)中所有的模块的函数。

```javascript
compilation.plugin('normal-module-loader', function(loaderContext, module) {
  // 这里是所以模块被加载的地方
  // 一个接一个，此时还没有依赖被创建
});
```
## seal
编译的封闭已经开始。

```javascript
compilation.plugin('seal', function() {
  // 你已经不能再接收到任何模块
  // 没有参数
});
```
## optimize
优化编译。

```javascript
compilation.plugin('optimize', function() {
  // webpack 已经进入优化阶段
  // 没有参数
});
```
## optimize-tree(chunks, modules) 异步
树的异步优化。


```
compilation.plugin('optimize-tree', function(chunks, modules) {

});
```
## optimize-modules(modules: Module[])
模块的优化。

```
compilation.plugin('optimize-modules', function(modules) {
  // 树优化期间处理模块数组
});
```
## after-optimize-modules(modules: Module[])
模块优化已经结束。
## optimize-chunks(chunks: Chunk[])
块的优化。

```
//这里一般只有一个块，除非你在配置中指定了多个入口

compilation.plugin('optimize-chunks', function(chunks) {
    // 这里一般只有一个块，
    // 除非你在配置中指定了多个入口
    chunks.forEach(function (chunk) {
        // Chunks have circular references to their modules
        chunk.modules.forEach(function (module){
            // module.loaders, module.rawRequest, module.dependencies, etc.
        });
    });
});
```
