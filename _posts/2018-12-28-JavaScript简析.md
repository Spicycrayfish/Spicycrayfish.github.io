---
layout:     post
title:      JavaScript简析(一)
subtitle:   个人收集的一些关于JavaScript的小技巧/注意点(涉及数据类型、语句、对象)，只收集了我以前没注意到的地方，所以不具备普适性
date:       2018-12-28
author:     Tank
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - 前端
    - JavaScript
---

> 本文首次发布于 [Tank's Blog](https://spicycrayfish.github.io/), 作者 [@谭轲](http://github.com/Spicycrayfish) ,转载请保留原文链接.



# 数据类型

#### 六种数据类型

原始类型：

* number
* string
* boolean
* null
* undefined

以及 对象类型：object (其中包括 Array、Date 等)





#### `JavaScript` 隐式转换

```javascript
let answer;
answer = 32 + 32; // 64
answer = '32' + 32; // '3232'
answer = '32' - 32; // 0
```



巧用 +/- 规则转换类型：

```javascript
num - 0   // 将 num 转换为数字
num + ''  // 将 num 转换为字符串
```



== 与 ===：

```javascript
'1.23' == 1.23;   null == undefined;    [1, 2] == [1, 2];
0 == false;     new Object() == new Object();
```

=== 需要注意的几个地方：

```javascript
null === null;    undefined === undefined;
NaN !== NaN;    new Object() !== new Object;
```





#### 包装对象

原始类型中，**number**、**string**、**boolean** 这三种原始类型都有对应的包装类型。以 **string** 为例：

```javascript
// 基本对象
var str = 'string';

// 包装对象
var strObj = new String('string')
```

基本类型不是对象类型 不应该有属性也不应该有方法，下面有几个问题：

```javascript
var str = 'string';

str.length    // 6  为什么会有 length 属性
str.t = 10    // 10 为什么可以赋值成功

str.t     // undefined  为什么取不到刚刚赋的属性值
```

解释：

在 `JavaScript` 中，当把一个基本类型尝试以对象的方式去使用它的时候（比如上述操作的访问 `legnth` 属性、增加 `t` 属性），`JavaScript` 会把这个基本类型转换为对应的包装类型对象，当完成这个操作之后（如访问完 `length` 属性、设置完 `t` 属性之后）这个临时对象会被销毁掉。所以上面操作中 `str.t = 10` 可以赋值成功，但马上就被销毁了，之后再访问该属性则会返回 `undefined`。

同样的 **number** 和 **boolean** 也是如此。





#### 类型检测

**typeof**、**instanceof**、**Object.prototype.toString**、**constructor**、**duck type**

最常用的是 `typeof`，适合于函数以及基本类型的判断：

```javascript
typeof 100      // "number"
typeof true     // "boolean"
typeof function   // "function"

typeof(undefined) // "undefined"
typeof new Object() // "object"
typeof [1, 2]   // "object"
typeof NaN      // "number"

typeof null     // "object"
```

判断对象类型常用 `instanceof`，它是基于原型链去判断的操作符：

```javascript
obj instanceof Object   // 左边对象的原型链上是否有右边这个构造函数的 prototype 属性

[1，2] instanceof Array === true;
new Object() instanceof Array === false;
```

> 注：不同 **window** 或 **iframe** 间的对象类型检测不能使用 **instanceof**

还可使用 `Object.prototype.toString` ：

```javascript
Object.prototype.toString.apply([]);    // "[object Array]"
Object.prototype.toString.apply(function(){}) // "[object Function]"
Object.prototype.toString.apply(null)   // "[object Null]"  IE6/7/8 返回 [object Object]
Object.prototype.toString.apply(undefined)  // "[object Undefined]"
```



总结：

* `typeof`：适合基本类型及 **function** 的检测，遇到 **null** 失效；
* `Object.prototype.toString`：适合内置对象和基本类型，遇到 **nul**l 和 **undefined** 失效（IE678返回 [object Object]）;
* `instanceof`：适合自定义对象，也可以用来检测原生对象，在不同的 **iframe** 和 **window** 间检测时失效。







# 语句

#### 块语句

常用于组合0~多个语句，块语句用一对花括号定义。

> 注意：没有块级作用域，所以下面两段代码是一样的：
>
> ```javascript
> for (var i = 0; i < 10; i++) {
>     console.log(i);
> }
> 
> {
>     var x = 1;
> }
> ```
>
> ```javascript
> var i = 0;
> for (; i < 10; i++) {
>     console.log(i)
> }
> 
> var x = 1;
> {
>     // ...
> }
> ```



#### var

`var a = b = 1;` 这段代码声明了两个变量，但是其中的 `b` 是一个全局变量，这样是隐式创建了一个全局变量。

```javascript
function foo() {
    var a = b = 1;
}
foo();

console.log(typeof a);  // 'undefined'
console.log(typeof b);  // 'number'
```



#### try catch

关于`try catch` 嵌套使用时要注意的问题：

没有`catch` 时会跳到最近的一层 `catch` 处理，跳出其所在块语之前会先执行 `finally` 。

```javascript
try {
    try {
        throw new Error('opps');
    }
    finally {
        console.log('finally');
    }
}
catch(ex) {
    console.error('outer', ex.message);
}

// 执行结果
// 'finally'
// 'outer' 'opps'
```



```javascript
try {
    try {
        throw new Error('opps');
    }
    catch(ex) {
        console.log('inner', ex.message);
    }
    finally {
        console.log('finally');
    }
}
catch(ex) {
    console.log('outer', ex.message);
}

// 执行结果
// ‘inner’ 'opps'
// 'finally'
```



```javascript
try {
    try {
        throw new Error('opps');
    }
    catch(ex) {
        console.log('inner', ex.message);
        throw ex
    }
    finally {
        console.log('finally');
    }
}
catch(ex) {
    console.log('outer', ex.message);
}

// 执行结果
// ‘inner’ 'opps'
// 'finally'
// 'outer' 'opps'
```



#### with(不建议使用)

`with` 语句可以修改当前的作用域：

```javascript
with ({x : 1}) {
    console.log(x)    // 1
}

with (document.forms[0]) {
    console.log(name.value);  // 相对于访问 document.forms[0].name.value
}
```

一般可以直接用变量来代替 不用这么麻烦...  不建议使用 `with`



#### 严格模式

严格模式是一种特殊的运行模式，它修复了部分语言上的不足，提供更强的错误检查，并增强安全性。

* 不允许使用 `with`

* 不允许未声明的变量被赋值

  *一般模式下直接给未声明的变量赋值相当于声明了一个全局变量*

* `arguments` 变为参数的静态副本

  ```javascript
  !function(a) {
      arguments[0] = 100;
      console.log(a);   // 100
  }(1);
  
  // 一般模式下定义一个函数并传参，其形参 a 与 arguments[0] 是有相互的绑定关系的
  // 上面代码中传入参数是1，但通过 arguments[0] 修改为了 100
  
  // 注意：如果没有传任何值，即 a 是 un defined，此时将 arguments[0] 赋值为100的话，
  // a 依然是 undefined，不会受 arguments[0] 的影响
  ```

  ```javascript
  !function(a) {
      'use strict';
      argument[0] = 100;
      console.log(a)    // 1
  }(1);
  ```

  如果传入的参数是对象 修改对象的属性仍然会相互影响

  ```javascript
  !function(a) {
      'use strict';
      argument[0].x = 100;
      console.log(a.x)    // 100
  }({x: 1});
  ```

* `delete` 接参数或函数名的话会报错

  ```javascript
  !function(a) {
      console.log(delete a) // false 删除失败
  }(1);
  
  !function(a) {
      'use strict';
      delete a;   // SyntaxError 报错
  }(1);
  ```

* `delete` 不可配置的属性也会报错

  ```javascript
  !function(a) {
      var obj = {};
      Object.defineProperty(obj, 'a', {
          configurable: false
      });
      console.log(delete obj.a);    // false 删除失败
  }(1);
  
  !function(a) {
      'use strict';
      var obj = {};
      Object.defineProperty(obj, 'a', {
          configurable: false
      });
      delete obj.a;   // TypeError 报错
  }(1);
  ```

* 对象字面量重复属性名报错

  ```javascript
  !function() {
      var obj = {x:1, x:2};
      console.log(obj.x);   // 2
  }();
  
  !function() {
      'use strict';
      var obj = {x:1, x:2}; // SyntaxError 报错
  }();
  ```

* 禁止八进制字面量

  ```javascript
  !function() {
      console.log(0123);    // 83
  }()
  
  !function() {
      'use strict';
      console.log(0123);    // SyntaxError 报错
  }()
  ```

* `eval`、`arguments` 变为关键字，不能作为变量、函数名

* `eval` 独立作用域

  ```javascript
  !function() {
      eval('var evalVal = 2;');
      console.log(typeof evalVal);  // number
  }
  
  !function() {
      'use strict';
      eval('var evalVal = 2;');
      console.log(typeof evalVal);  // undefined
  }
  ```

* 一般函数调用时（不是对象的方法调用，也不使用 apply/call/bind 等修改 this）this 指向 null，而不是全局对象

* 若使用 apply/call，当传入 null 或 undefined 时，this 指向 null 或 undefined，而不是全局对象

* 试图修改不可写属性（writable = false），在不可扩展的对象上添加属性时报 TypeError，而不是忽略

* arguments.caller，arguments.callee 被禁用







# 对象

#### 原型链

```javascript
function foo() {}

// 默认带一个 prototype 属性，该属性是一个对象属性
foo.prototype.z = 3;

// 通过 new 创建的对象，其原型会指向构造器的 prototype 属性
var obj = new foo();
obj.x = 1;
obj.y = 2;

typeof obj.toString;  // 'function'
'z' in obj;   // true
obj.hasOwnProperty('z');  // false
```

当一个对象上没有我们访问的属性，会通过原型链向上查找，一直找到 `Object.prototype`，`Object.prototype` 是 `null`，找到这里还没有的话就会返回 `undefined`；赋值则不会向原型链上去查找。





#### 属性操作

`for in` 可遍历对象的属性，但也会枚举其原型链上的属性，当我们不需要其原型链上的属性时可以加一个判断来筛选：

```javascript
var o = {x:1, y:2, z:3};

var obj = Object.create(o);
obj.a = 4;

var key;
for (key in obj) {
    if(obj.hasOwnProperty(key)) {
        console.log(key); // a
    }
}
```





#### 属性 getter/setter 方法

```javascript
var man = {
    name: 'kobe',
    get age() {
        return new Date().getFullYear() - 1978;
    },
    set age(val) {
        console.log('age can\'t be set to ' + val);
    }
}
```

> 注意，get/set 方法与原型链之间关系较复杂

```javascript
function foo() {};

Object.defineProperty(foo.prototype, 'z',
     {get: function(){ return 1; }});

var obj = new foo();

obj.z;    // 1
obj.z = 10;
obj.z   // 1
// 此时该对象原型链上的 z 属性是get方法定义的，不能通过直接赋值的方法修改该对象的 z 属性值

Object.defineProperty(obj, 'z', {
    value: 100, configurable: true
});
obj.z;    // 100
delete obj.z;
obj.z;    // 1
```





#### 属性标签

使用 `getOwnPropertyDescriptor` 查看一个对象的某属性拥有哪些标签：

```javascript
// 该函数接收两个参数：第一个参数为要查询的对象，第二个参数为要查询的属性名
Object.getOwnPropertyDescriptor({pro : true}, 'pro');
//  Object {value: true, writable: true, enumerable: true, configurable: true}

Object.getOwnPropertyDescriptor({pro : true}, 'a');
// undefined
```

* writable：是否可写
* enumerable：是否可枚举/遍历
* configurable：是否可再修改/delete

```javascript
var person = {};
Object.defineProperty(person, 'name', {
    configurable: false,
    writable: false,
    enumerable: true,
    value: 'tank'
});

person.name;  // tank
person.name = 1;
person.name   // tank
delete person.name; // false
```

一次定义多个属性：

```javascript
Object.defineProperties(person, {
    salary: {value:50000, enumerable:true, writable:true},
    promote: {
        set: function(level) {
            this.salary *= 1 + level*0.1;
        }
    }
})
```

`configurabel` 与 `writable` 的影响：


|  | configurable: true<br />writable: true | configurable: true<br />writable: false | configurable: false<br />writable: true | configurable: false<br />writable: false |
| ------ | :----- | ------ | ------ | ------ |
| 修改属性的值 | Yes | Yes | Yes | No |
| 通过属性赋值修改属性的值 | Yes | No | Yes | No |
| delete该属性返回true | Yes | Yes | No | No |
| 修改 getter/setter 方法 | Yes | Yes | No | No |
| 修改属性标签(除了writable从true修改为false总是允许的) | Yes | Yes | No | No |





#### 对象标签

* proto：new构造器生成的对象，该标签指向构造器的 prototype 属性；一般指向 Object.prototype。

* class：对象的类

* extensible：对象是否可扩展

  ```javascript
  var obj = {x: 1, y: 2};
  Object.isExtensible(obj);   // true
  Object.preventExtensions(obj);
  Object.isExtensible(obj);   // false
  obj.z = 1;    // undefined, add new property failed
  
  Object.seal(obj);
  Object.getOwnPropertyDescriptor(obj, 'x');
  // Object(value: 1, writable: true, enumerable: true, configurable: false)
  Object.isSealed(obj); // true
  
  Object.freeze(obj);
  Object.getOwnPropertyDescriptor(obj, 'x');
  // Object(value: 1, writable: false, enumerable: true, configurable: false)
  Object.ifFrozen(obj); // true
  ```





#### 序列化与自定义序列化

使用 `JSON.stringify()` 进行序列化

> 注意：1. 若属性的值为 `undefined`，该属性则不会出现在序列化之后的字符串结果中；2. 若属性值为 `NaN` 、`Infinity`，字符串结果会将其转换为 `null`；3. 若属性值是时间，则在结果中会转换为UCT的时间格式。

示例代码：

```javascript
var obj = {val: undefined, a: NaN, b: Infinity, c: new Date()};
JSON.stringify(obj);
// "{"a":null,"b":null,"c":"2018-11-20T14:15:24.910Z"}"
```



使用 `toJSON` 函数实现自定义序列化：

```javascript
var obj = {
    x: 1,
    y: 2,
    o: {
        o1: 3,
        o2: 4,
        toJSON: function() {
            return this.o1 + this.o2;
        }
    }
};

JSON.stringify(obj);
// "{"x":1,"y":2,"o":7}"
```

