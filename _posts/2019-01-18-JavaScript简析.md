---
layout:     post
title:      JavaScript简析(二)
subtitle:   个人收集的一些关于JavaScript的小技巧/注意点(涉及数组、函数、面向对象编程、正则表达式)，只收集了我以前没注意到的地方，所以不具备普适性
date:       2019-01-18
author:     Tank
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - 前端
    - JavaScript
---

> 本文首次发布于 [Tank's Blog](https://spicycrayfish.github.io/), 作者 [@谭轲](http://github.com/Spicycrayfish) ,转载请保留原文链接.


# 数组

### 稀疏数组

稀疏数组并不含有从0开始的**连续**索引，一般 `length` 属性值比实际元素个数大。

```javascript
var arr1 = [undefined];
var arr2 = new Array(1);  // 稀疏数组

0 in arr1;    // true
0 in arr2;    // false

var arr = [,,];   // 稀疏数组
0 in arr;   // false
```





### every & some

```javascript
var arr = [1, 2, 3, 4, 5];
arr.every(function(x) {
    return x < 10;
});   // true

arr.every(function(x) {
    return x < 3;
});   // false

arr.some(function(x) {
    return x === 3;
});   // true

arr.some(function(x) {
    return x === 10;
});   // false
```





### reduce & reduceRight

`reduceRight` 与 `reduce` 的区别在于 `reduceRight` 从右开始操作数组元素

```javascript
var arr = [1, 2, 3];
var sum = arr.reduce(function(x, y) {
    return x + y;
}, 0);    // 6

arr = [3, 6, 9];
var max = arr.reduce(function(x, y) {
    return x > y ? x : y;
});     // 9
```





### indexOf & lastIndexOf

`indexOf` 接收两个参数，第一个参数为要查找的值，第二个参数为查找数组的起始位置：

```javascript
var arr = [1, 2, 3, 2, 1];

arr.indexOf(1, 1);  // 4
arr.indexOf(1, -3); // 4
arr.indexOf(2, -1); // -1

arr.lastIndexOf(2); // 3
arr.lastIndexOf(2, -2); // 3
arr.lastIndexOf(2, -3); // 1
```







# 函数

函数是一块 *JavaScript* 代码，被定义一次，但可执行和调用多次。JS中的函数也是对象，所以JS函数可以像其他对象那些操作和传递，所以也常叫JS中的函数为函数对象。

重点：

* this
* arguments
* 作用域
* 不同调用方式
* 不同创建方法





### this

全局的 `this` (浏览器)：`this === window`



对象原型链上的 `this`：

```javascript
var o = {f: function(){ return this.a + this.b; }};
var p = Object.create(o);

p.a = 1;
p.b = 4;

p.f();  // 5
```



构造器中的 `this`：

```javascript
function MyClass() {
    this.a = 36;
}

var o = new MyClass();
o.a;  // 36

function C2() {
    this.a = 24;
    return {a:20};
}

o = new C2();
o.a;  // 20
```

> 注：使用 `new` 关键字创建对象，其构造器如果没有返回对象或者返回的是基本类型，则会将 `this` 作为返回值；但如果构造器 `return` 返回的是对象，那么它会将 `return` 的对象作为返回值。



`call` / `apply` 与 `this`：

```javascript
function add(c, d) {
    return this.a + this.b + c + d;
}

var o = {a:1, b:3};

add.call(o, 5, 7);    // 1 + 3 + 5 + 7 = 16
add.apply(o, [10, 20]); // 1 + 3 + 10 + 20 = 34

function bar() {
    console.log(Object.prototype.toString.call(this));
}

bar.call(7);    // "[object Number]"
```



`bind` 方法与 `this`：

```javascript
function f() {
    return this.a;
}

var g = f.bind({a: 'test'});
g();  // test

var o = {a:37, f:f, g:g};
console.log(o.f(), o.g());  // 37, test
```





### arguments

```javascript
function foo(x, y, z) {
    arguments.length; // 2
    arguments[0];   // 1
    arguments[0] = 10;
    x;  // change to 10，传参后形参和 arguments 会有绑定关系，严格模式下仍然是 1
    
    arguments[2] = 100;
    z;  // still undefined，未传参则失去绑定关系
    arguments.callee === foo; // true 严格模式下不可使用
}

foo(1, 2);
foo.length;   // 3
foo.name;   // "foo"

// foo.name  函数名
// foo.length  函数形参个数
// arguments.length  实参个数
```



apply / call 方法(浏览器)：

```javascript
function foo(x, y) {
    console.log(x, y, this);
}

foo.call(100, 1, 2);  // 1, 2, Number(100)
foo.apply(true, [3, 4]);  // 3, 4, Boolaen(true)
foo.apply(null);    // undefined, undefined, window(严格模式下是 null)
foo.apply(undefined);   // undefined, undefined, window(严格模式下是 undefined)
```



bind 方法：

```javascript
this.x = 9;
var module = {
    x: 81,
    getX: function() { return this.x; }
};

module.getX();  // 81
var getX = module.getX;
getX();   // 9

var boundGetX = getX.bind(module);
boundGetX();  // 81
```



bind 与 currying：

```javascript
function add(a, b, c) {
    return a + b + c;
}

var func = add.bind(undefined, 100);  // 将 100 绑定为 add 函数的第一个参数
func(1, 2); // 103

var func1 = func.bind(undefined, 200);  // 将 200 绑定为 func 函数的第一个参数，即 add 函数的第二个参数
func1(10);  // 310
```





### 作用域

全局作用域、函数作用域、eval作用域：

```javascript
var a = 100;  // 全局作用域

(function() {
    var b = 200;  // 函数作用域
})();

eval('var c = 300');  // eval作用域
```

> 注：`JavaScript` 中没有块级作用域
>
> ```javascript
> for (var item in {a:1, b:2}) {
>     console.log(item);
> }
> console.log(item);  // item 在循环体外任可被访问
> ```



作用域链需要注意的是，使用 `new Function()` 或者 `Funciton()` 构造器并用字符串的方式调用而构造的函数，无法访问到构造器定义位置的变量：

```javascript
var i = 1;
var func = new Function('console.log(typeof i);');
func(); // undefined
```







# 附：ES3执行上下文

对于全局作用域会有一个全局的执行上下文，同一个函数在被调用多次的时候，每次都有不同的执行上下文。



###执行上下文

**执行上下文** (`Execution Context`) 类似于一个栈的结构：

```javascript
console.log('EC0');
function funcEC1() {
    console.log('EC1');
    var funcEC2 = function() {
        console.log('EC2');
        var funcEC3 = function() {
            console.log('EC3');
        };
        funcEC3();
    }
    funcEC2();
}
funcEC1();
// E0 E1 E2 E3
```

一开始会进入全局的 **EC0**，当调用 `funcEC1` 时控制权会从 **EC0** 到 **EC1** 的执行上下文，以此类推进入 **EC2**、**EC3**；在 **EC3** 调用结束后会退回到 **EC2**、**EC1**，`funcEC1` 执行结束后退回到全局上下文 **EC0**，知道整个代码执行完毕。





### 变量对象

JS解释器如何找到我们定义的函数和变量？

**变量对象** (`Variable Object`) 是一个抽象概念中的“对象”，它用于存储执行上下文中的：

1. 变量
2. 函数声明
3. 函数参数

```javascript
activeExcutionContext = {
    VO: {
        data_var,
        data_func_declaration,
        data_func_arguments
    }
};

// 全局作用域下也会有一个变量对象
GlobalContextVO     (VO === this === global)
```



变量对象示例：

```javascript
var a = 10;
function test(x) {
    var b = 20;
}
test(30);

// 对于以上代码，其变量对象如下：
VO(globalContext) = {
    a: 10,
    test: <ref to function>
};

VO(test functionContext) = {
    x: 30,
    b: 20
};
```



在 `JavaScript` 第一行代码就可以调用 `Math` 、 `String` 等方法，在浏览器中也可以拿到 `window`，这是因为在全局上下文中有这么一个变量对象，在代码执行前，JS引擎会把全局的类、方法等初始化到变量对象中：

```javascript
VO(globalContext) === [[global]];

[[global]] = {
    Math: <...>,
    String: <...>,
    isNaN: function() {[Native Code]}
};

GlobalContextVO   (VO === this === global)

String(10);   // [[global]].String(10);
window.a = 10;  // [[global]].window.a = 10;
this.b = 20;  // [[global]].b = 20;
```



函数中有一个概念叫激活对象，在函数调用时会有一个特殊的 `arguments`，在初始化阶段时会被放入AO对象即激活对象；初始化 `arguments` 之后，这个AO对象又会被叫做VO对象，会和全局的VO一样来进行其它的一些初始化（函数形参、变量声明、函数声明）

```javascript
VO(functionContext) === AO;

AO = {
    arguments: <Arg0>
};
```



对于函数来讲，VO和AO是一个对象，其VO或AO分为两个阶段，第一个阶段是变量初始化阶段，做一些 `arguments` 的初始化，以及变量声明和函数声明；

```javascript
function test(a, b) {
    var c = 10;
    function d() {}
    var e = function _e() {};
    (function x() {});
    b = 20;
}
test(10);

// 以上代码的初始化阶段的VO如下
AO(test) = {
    a: 10,
    b: undefined,
    c: undefined,
    d: <ref to func "d">,
    e: undefined
}
```

VO 按照如下顺序填充：

1. 函数参数（若未传入，初始化该参数值为 `undefined`）
2. 函数声明（若发生命名冲突，会覆盖）
3. 变量声明（初始化变量值为 `undefined`，若发生命名冲突，会忽略）

> 注：函数表达式不会影响VO

第二个阶段是代码执行阶段，：

```javascript
function test(a, b) {
    var c = 10;
    function d() {}
    var e = function _e() {};
    (function x() {});
    b = 20;
}
test(10);

// 代码执行阶段后的VO如下
AO(test) = {
    a: 10,
    b: 20,
    c: 10,
    d: <ref to func "d">,
    e: function _e() {};
}
```



测试代码示例：

```javascript
alert(x); // function

var x = 10;
alert(x); // 10
x = 20;

function x() {}
alert(x);   // 20

if (true) {
    var a = 1;
} else {
    var b = true;
}

alert(a); // 1
alert(b); // undefined
```

解析：

* 在初始化阶段，首先没有函数参数的初始化；再查看函数声明，只有一个函数 `x` 的声明；接下来是变量声明，分别是 `x`、`a`、`b`，其中 `x` 跟函数声明冲突了，但是会选择忽略，而JS中没有块级作用域，`a`、`b` 提前处理，所以 `a` 和 `b` 则初始化为 `undefined`。
* 在代码执行阶段，第一行代码的 `x` 为初始值，即一个函数对象；接下来 `x` 依次被赋值为10、20；由于函数声明提前处理了，在代码执行阶段 `function x() {}` 这行代码可以忽略掉，所以 `x` 的值依然为20；后面代码中执行 `a` 的赋值语句







# JavaScript面对对象编程

继承的示例代码：

```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
}

Person.prototype.hi = function() {
    console.log("hi, my name is" + this.name + ", i am" + this.age + "years old now")
};

Person.prototype.LEGS_NUM = 2;
Person.prototype.ARMS_NUM = 2;
Person.prototype.walk = function() {
    console.log(this.name + "is walking");
};

// 创新一个继承自 Person 的 Student 类
function Student(name, age, className) {
    Person.call(this, name, age);
    this.className = className;
}
Student.prototype = Object.create(Person.prototype)
Student.prototype.constructor = Student;

Student.prototype.hi = function() {
    console.log("I'm from" + this.className)
}
Student.prototype.learn = function(subject) {
    console.log("I'm learning" + subject);
}
```





### 改变prototype

```javascript
var bosn = new Student('Bosn', 24, 'Class 5, Grade 3');

// 添加或删除一些 prototype 属性会影响到已创建的实例
Student.prototype.x = 101;
bosn.x;   // 101

// 直接修改 prototype，不会影响已创建的实例，但会影响到后面创建的实例
Student.prototype = {y : 2};
bosn.y;   // undefined
bosn.x;   // 101

var carol = new Student('Carol', 24, 'Class 5, Grade 3');
carol.x;  // undefined
carol.y;  // 2
```







# JavaScript正则表达式

正则基础：


字符|匹配项|示例
--|:--:|--:
.|任意字符（除换行符外：\n, \r, \u2028 or \u2029）|/.../.test('1a@');
\d|数字0-9|/\d\d\d/.test('123');
\D|非\d，即不是数字0-9的字符|/\D\D\D/.test('a@!');
\w|数字0-9，或字母a-z及A-Z，或下划线|/\w\w\w\w/.test('aB1_');
\W|非\w|/\W\W\W/.test('!@#');
\s|空格符，TAB，换页符，换行符|/\sabc/.test('abc');
\S|非\s|

\t  \r  \n  \v  \f  分别对应：tab  回车  换行   垂直制表符  换页符



范围符号：

| 符号   |     含义     |                             示例 |
| ------ | :----------: | -------------------------------: |
| [...]  |     字符     | [a-z]  /  [0-9]  /  [A-Z0-9a-z_] |
| [^...] | 字符范围以外 |                                  |
| ^      |     行首     |                              ^Hi |
| $      |     行尾     |                           world$ |
| \b     | 零宽单词边界 |                             \bno |
| \B     |     非\b     |                                  |



重复：

| 符号                 |                      含义                      |                                                   示例 |
| -------------------- | :--------------------------------------------: | -----------------------------------------------------: |
| x*   x+              |       重复次数>=0   重复次数>0；贪婪算法       |                              abc*将匹配ab、abc、abcccc |
| x*?  x+?             |             同x*   x+；非贪婪算法              | abc*?将在字符串abccccc中将匹配ab<br />abc+?则匹配abc。 |
| x?                   |                  出现0次或1次                  |                                                        |
| x\|y                 |                     x或者y                     |                                                        |
| x{n}  x{n,}  x{n, m} | 重复n次，重复>=n次，重复次数x满足：n <= x <= m |                                                        |



三个flag：

* global：全局检索
* ignoreCase：不区分大小写
* multiline：分行、跨行检索

```javascript
/abc/gim.test('ABC');   // true
RegExp('abc', 'mgi');
```

其中 `gim` 分别代表 `global`、`ignoreCase`、`multiline`，顺序无关



RegExp对象常见属性：

* global：是否为全局检索
* ignoreCase：是否不区分大小写
* multiline：是否跨行检索
* source：正则的内容

```javascript
/abc/g.global;  // true
/abc/i.ignoreCase;  // true
/abc/g.multiline; // false
/abc/g.source;  // 'abc'
```



RegExp对象方法：

* compile：改变正则属性
* exec：返回匹配内容
* test：判断是否匹配
* toString

```javascript
/abc/.exec('abcdef'); // "abc"
/abc/.test('abcde');  // true
/abc/.toString(); // "/abc/"

var reg = /abc/;
reg.compile('def');
reg.test('def');  // true
```



string类型与正则相关的方法：

```javascript
// String.prototype.search
"abcabcdef".search(/(abc)\1/);    // 0

// String.prototype.replace
"aabbbbcc".replace(/b+?/, "1");   // aa1bbbcc

// String.prototype.match
"aabbbbcc".match(/b+/);   // ["bbbb"]
"aabbbbccbbaa".match(/b+/g);  // ["bbbb", "bb"]

// String.prototype.split
"aabbbbccbbaa".split(/b+/); // ["aa", "cc", "aa"]
```

