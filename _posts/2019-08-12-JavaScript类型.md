---
layout:     post
title:      JavaScript类型
subtitle:   关于JavaScript类型的小事项，包括Number类的精度比较问题，Object的装箱转换和拆箱转换
date:       2019-08-12
author:     Tank
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 前端
    - JavaScript
---

> 本文首次发布于 [Tank's Blog](https://spicycrayfish.github.io/), 作者 [@谭轲](http://github.com/Spicycrayfish) ,转载请保留原文链接.



# 引言

文章内容总结自 [**极客时间**](https://time.geekbang.org/) 的订阅专栏 [**重学前端**](https://time.geekbang.org/column/intro/154)，节选了其中我觉得不太熟悉的地方。



## Number

非整数的 Number 类型无法用 == (===也不行) 来比较

```javascript
console.log( 0.1 + 0.2 == 0.3);
```

输出结果为 false，因为等式两边相差一个微小的值，正确比较方法是使用 JavaScript 提供的最小精度值：

```javascript
console.log( Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON);
```



## Object

 运算符提供了装箱操作，它会根据基础类型构造一个临时对象，使得我们能在基础类型上调用对应对象的方法。

如：

```javascript
console.log("abc".charAt(0)); //a
```

`"abc"` 是基础类型 `String`，但是可以调用对象方法 `charAt`


JavaScript 中的几个基本类型，都在对象类型中有一个亲戚，即都可以根据基础类型构造临时对象并调用对象方法，这几个类型是：

* Number
* String
* Boolean
* Symbol


在原型上添加方法，都可以应用于基本类型，比如以下代码，在 Symbol 原型上添加了 hello 方法，在任何 Symbol 类型变量都可以调用：

```javascript
Symbol.prototype.hello = () => console.log("hello");

var a = Symbol("a");
console.log(typeof a); //symbol，a 并非对象
a.hello(); //hello，有效

```



## 装箱转换

每一种基本类型 Number、String、Boolean、Symbol 在对象中都有对应的类，所谓装箱转换，正是把基本类型转换为对应的对象。

使用内置的 Object 函数，我们可以在 JavaScript 代码中显式调用装箱能力：

```javascript
var symbolObject = Object(Symbol("a"));

console.log(typeof symbolObject); //object
console.log(symbolObject instanceof Symbol); //true
console.log(symbolObject.constructor == Symbol); //true

```

用 `console.log` 看一下 `symbolObject` 的 `type of`，它的值是 `object`，我们使用 `symbolObject instanceof` 可以看到，它是 `Symbol` 这个类的实例，我们找它的 `constructor` 也是等于 `Symbol` 的，所以我们无论从哪个角度看，它都是 `Symbol` 装箱过的对象



每一类装箱对象皆有私有的 Class 属性，这些属性可以用 Object.prototype.toString 获取：

```javascript
var symbolObject = Object(Symbol("a"));

console.log(Object.prototype.toString.call(symbolObject)); //[object Symbol]
```





## 拆箱转换

在 JavaScript 标准中，规定了 `ToPrimitive` 函数，它是对象类型到基本类型的转换（即，拆箱转换）。对象到 String 和 Number 的转换都遵循“先拆箱再转换”的规则。通过拆箱转换，把对象变成基本类型，再从基本类型转换为对应的 String 或者 Number。

拆箱转换会尝试调用 `valueOf` 和 `toString` 来获得拆箱后的基本类型。如果 `valueOf` 和 `toString` 都不存在，或者没有返回基本类型，则会产生类型错误 `TypeError`：

```javascript
  var o = {
      valueOf : () => {console.log("valueOf"); return {}},
      toString : () => {console.log("toString"); return {}}
  }

  o * 2
  // valueOf
  // toString
  // TypeError

```

我们定义了一个对象 o，o 有 valueOf 和 toString 两个方法，这两个方法都返回一个对象，然后我们进行 o*2 这个运算的时候，你会看见先执行了 valueOf，接下来是 toString，最后抛出了一个 TypeError，这就说明了这个拆箱转换失败了。

到 String 的拆箱转换会优先调用 toString。我们把刚才的运算从 o*2 换成 String(o)，那么你会看到调用顺序就变了：

```javascript
var o = {
    valueOf : () => {console.log("valueOf"); return {}},
    toString : () => {console.log("toString"); return {}}
}

String(o)
// toString
// valueOf
// TypeError

```



在 ES6 之后，还允许对象通过显式指定 @@toPrimitive Symbol 来覆盖原有的行为：

```javascript
var o = {
    valueOf : () => {console.log("valueOf"); return {}},
    toString : () => {console.log("toString"); return {}}
}

o[Symbol.toPrimitive] = () => {console.log("toPrimitive"); return "hello"}


console.log(o + "")
// toPrimitive
// hello

```




