# JavaScript 中 bind 的使用与实现 `ES5`
```javascript
var altwrite = document.write;
altwrite("hello");
```
上面这个例子会提示 `document is not undefind`, 例子调用时的 this 已经改变了, 从 document => window 如果想让方法跑通必须把 this 指回到 document, 如 `altwrite.bind(document)('hello')` `altwrite.call(document, 'hello')`

## 绑定函数
`bind` 的作用就是**创造一个函数**, 不管怎么调用都保持 `this` 不变. 常见的错误就像上面的例子一样，将方法从对象中拿出来，然后调用，并且希望this指向原来的对象。如果不做特殊处理，一般会丢失原来的对象。
```javascript
this.num = 9
var foo = {
  num: 1,
  getNum: function () {
    return this.num
  }
}
foo.getNum() // 1
var temp = foo.getNum
temp() // 9
var bindTemp = temp.bind(foo)
bindTemp() // 1
```
## 偏函数
> 在计算机科学中，局部应用是指固定一个函数的一些参数，然后产生另一个更小元的函数。
> 什么是元？元是指函数参数的个数，比如一个带有两个参数的函数被称为二元函数。
偏函数和柯里化的区别
> 柯里化是将一个多参数函数转换成多个单参数函数，也就是将一个 n 元函数转换成 n 个一元函数。
> 局部应用则是固定一个函数的一个或者多个参数，也就是将一个 n 元函数转换成一个 n - x 元函数。

Case1
```javascript
function add(a, b) {
    return a + b;
}

var addOne = add.bind(null, 1); // a = 1

addOne(2) // 3, b = 2
```
Case2
```javascript
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// 预定义参数37
var leadingThirtysevenList = list.bind(undefined, 37);

var list2 = leadingThirtysevenList(); // [37]
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```
## 和 setTimeout/setInterval 一起使用
`setTimeout` 的 `this` 一般指向 `window` 或者 `global` 对象, 使用继承方法时需要把 `this` 指向实例
```javascript
function Bloomer() {
  this.petalCount = Math.ceil(Math.random() * 12) + 1;
}

// 1秒后调用declare函数
Bloomer.prototype.bloom = function() {
  window.setTimeout(this.declare.bind(this), 1000); // 把 window 指向替换成 this
};

Bloomer.prototype.declare = function() {
  console.log('我有 ' + this.petalCount + ' 朵花瓣!');
};

var obj = new Bloomer()
obj.bloom() //
```
## 绑定函数作为构造函数
绑定函数也适用于使用new操作符来构造目标函数的实例。当使用绑定函数来构造实例，**注意：this会被忽略，但是传入的参数仍然可用。**
```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return this.x + ',' + this.y;
};

var p = new Point(1, 2);
p.toString(); // '1,2'

var YAxisPoint = Point.bind(null, 0/*x*/); // 第一个参数为 null 不改变 this 指向, 但是可以做预先参数赋值. YAxisPoint把 Point 作为它的构造函数接收了

var axisPoint = new YAxisPoint(5);
axisPoint.toString(); // '0,5'

axisPoint instanceof Point; // true
axisPoint instanceof YAxisPoint; // true
new Point(17, 42) instanceof YAxisPoint; // true

```
Point和YAxisPoint共享原型

## 实现
通过给目标函数指定作用域简单实现
```javascript
Function.prototype.bind = function (context) {
  self = this;  //保存this，即调用bind方法的目标函数
  return function () {
    return self.apply(context, arguments);
  };
};
```
更加健壮的 `bind`
```javascript
Function.prototype.bind = function (context) {
  var args = Array.prototype.slice.call(arguments, 1),
  self = this;
  return function () {
    var innerArgs = Array.prototype.slice.call(arguments);
    var finalArgs = args.concat(innerArgs);
    return self.apply(context, finalArgs);
  };
};
```
作为构造函数
```javascript
Function.prototype.bind = function (context) {
  var args = Array.prototype.slice.call(arguments, 1),
  F = function(){},
  self = this,
  bound = function () {
    var original = Array.prototype.slice.call(arguments)
    var final = args.concat(original) // 合并两处参数
    return self.apply((this instanceof F ? this : context), final)
  }

  F.prototype = self.prototype;
  bound.prototype = new F();
  retrun bound;
}
```
