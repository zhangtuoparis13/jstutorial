---
title: 面向对象编程概述
layout: page
category: oop
date: 2012-12-28
modifiedOn: 2014-02-04
---

## 对象是什么？

“面向对象编程”（Object Oriented Programming，缩写为OOP）是目前主流的编程范式。它的核心思想是将真实世界中各种复杂的关系，抽象为一个个对象，然后由对象之间的分工与合作，完成对真实世界的模拟。

传统的过程式编程（procedural programming）由一系列函数或一系列指令组成，而面向对象编程的程序由一系列对象组成。每一个对象都是功能中心，具有明确分工，可以完成接受信息、处理数据、发出信息等任务。因此，面向对象编程具有灵活性、代码的可重用性、模块性等特点，容易维护和开发，非常适合多人合作的大型软件项目。

那么，“对象”（object）到底是什么？

我们从两个层次来理解。

**（1）“对象”是单个实物的抽象。**

一本书、一辆汽车、一个人都可以是“对象”，一个数据库、一张网页、一个与远程服务器的连接也可以是“对象”。当实物被抽象成“对象”，实物之间的关系就变成了“对象”之间的关系，从而就可以模拟现实情况，针对“对象”进行编程。

**（2）“对象”是一个容器，封装了“属性”（property）和“方法”（method）。**

所谓“属性”，就是对象的状态；所谓“方法”，就是对象的行为（完成某种任务）。比如，我们可以把动物抽象为animal对象，“属性”记录具体是那一种动物，“方法”表示动物的某种行为（奔跑、捕猎、休息等等）。

虽然不同于传统的面向对象编程语言，但是JavaScript具有很强的面向对象编程能力。本章介绍JavaScript如何进行“面向对象编程”。

## 构造函数

“面向对象编程”的第一步，就是要生成“对象”。前面说过，“对象”是单个实物的抽象。通常需要一个模板，表示某一类实物的共同特征，然后“对象”根据这个模板生成。

典型的面向对象编程语言（比如C++和Java），存在“类”（class）这个概念。所谓“类”就是对象的模板，对象就是“类”的实例。但是，JavaScript语言的对象体系，不是基于“类”的，而是基于构造函数（constructor）和原型链（prototype）。这个小节介绍构造函数。

JavaScript语言使用构造函数（constructor）作为对象的模板。所谓“构造函数”，就是专门用来生成“对象”的函数。它提供模板，描述对象的基本结构。一个构造函数，可以生成多个对象，这些对象都有相同的结构。

构造函数是一个正常的函数，但是它的特征和用法与普通函数不一样。下面就是一个构造函数。

```javascript
var Vehicle = function () {
  this.price = 1000;
};
```

上面代码中，`Vehicle`就是构造函数，它提供模板，用来生成车辆对象。为了与普通函数区别，通常将构造函数的名字的第一个字母大写。

构造函数的特点有两个。

- 函数体内部使用了`this`关键字，代表了所要生成的对象实例。
- 生成对象的时候，必需用`new`命令，调用`Vehicle`函数。

## new命令

### 基本用法

`new`命令的作用，就是执行构造函数，返回一个实例对象。

```javascript
var Vehicle = function (){
  this.price = 1000;
};

var v = new Vehicle();
v.price // 1000
```

上面代码通过`new`命令，让构造函数`Vehicle`生成一个实例对象，保存在变量`v`中。这个新生成的实例对象，从构造函数`Vehicle`继承了`price`属性。在`new`命令执行时，构造函数内部的`this`，就代表了新生成的实例对象，`this.price`表示实例对象有一个`price`属性，它的值是1000。

使用`new`命令时，根据需要，构造函数也可以接受参数。

```javascript
var Vehicle = function (p) {
  this.price = p;
};

var v = new Vehicle(500);
```

`new`命令本身就可以执行构造函数，所以后面的构造函数可以带括号，也可以不带括号。下面两行代码是等价的。

```javascript
var v = new Vehicle();
var v = new Vehicle;
```

一个很自然的问题是，如果忘了使用new命令，直接调用构造函数会发生什么事？

这种情况下，构造函数就变成了普通函数，并不会生成实例对象。而且由于下面会说到的原因，this这时代表全局对象，将造成一些意想不到的结果。

```javascript
var Vehicle = function (){
  this.price = 1000;
};

var v = Vehicle();
v.price
// Uncaught TypeError: Cannot read property 'price' of undefined

price
// 1000
```

上面代码中，调用Vehicle构造函数时，忘了加上new命令。结果，price属性变成了全局变量，而变量v变成了undefined。

因此，应该非常小心，避免出现不使用new命令、直接调用构造函数的情况。为了保证构造函数必须与new命令一起使用，一个解决办法是，在构造函数内部使用严格模式，即第一行加上`use strict`。

```javascript
function Fubar(foo, bar){
  "use strict";

  this._foo = foo;
  this._bar = bar;
}

Fubar()
// TypeError: Cannot set property '_foo' of undefined
```

上面代码的`Fubar`为构造函数，`use strict`命令保证了该函数在严格模式下运行。由于在严格模式中，函数内部的`this`不能指向全局对象，默认等于`undefined`，导致不加`new`调用会报错（JavaScript不允许对`undefined`添加属性）。

另一个解决办法，是在构造函数内部判断是否使用`new`命令，如果发现没有使用，则直接返回一个实例对象。

```javascript
function Fubar(foo, bar){
  if (!(this instanceof Fubar)) {
    return new Fubar(foo, bar);
  }

  this._foo = foo;
  this._bar = bar;
}

Fubar(1, 2)._foo // 1
(new Fubar(1, 2))._foo // 1
```

上面代码中的构造函数，不管加不加new命令，都会得到同样的结果。

### new命令的原理

使用new命令时，它后面的函数调用就不是正常的调用，而是被new命令控制了。内部的流程是，先创造一个空对象，作为上下文对象，赋值给函数内部的this关键字。也就是说，this指的是一个新生成的空对象，所有针对this的操作，都会发生在这个空对象上。

构造函数之所以叫“构造函数”，就是说这个函数的目的，就是操作上下文对象（即this对象），将其“构造”为需要的样子。如果构造函数的return语句返回的是对象，new命令会返回return语句指定的对象；否则，就会不管return语句，返回构造后的上下文对象。

```javascript
var Vehicle = function (){
  this.price = 1000;
  return 1000;
};

(new Vehicle()) === 1000
// false
```

上面代码中，Vehicle是一个构造函数，它的return语句返回一个数值。这时，new命令就会忽略这个return语句，返回“构造”后的this对象。

但是，如果return语句返回的是一个跟this无关的新对象，new命令会返回这个新对象，而不是this对象。这一点需要特别引起注意。

```javascript
var Vehicle = function (){
  this.price = 1000;
  return { price: 2000 };
};

(new Vehicle()).price
// 2000
```

上面代码中，构造函数Vehicle的return语句，返回的是一个新对象。new命令会返回这个对象，而不是this对象。

new命令简化的内部流程，可以用下面的代码表示。

```javascript
function _new(/* constructor, param, ... */) {
  var args = [].slice.call(arguments);
  var constructor = args.shift();
  var context = Object.create(constructor.prototype);
  var result = constructor.apply(context, args);
  return (typeof result === 'object' && result != null) ? result : context;
}

var actor = _new(Person, "张三", 28);
```

## instanceof运算符

`instanceof`运算符返回一个布尔值，表示一个对象是否由某个构造函数创建。

```javascript
var v = new Vehicle();
v instanceof Vehicle // true
```

`instanceof`运算符的左边是实例对象，右边是构造函数。

在JavaScript之中，只要是对象，就有对应的构造函数。因此，`instanceof`运算符可以用来判断值的类型。

```javascript
[1, 2, 3] instanceof Array // true
({}) instanceof Object // true
```

上面代码表示数组和对象则分别是`Array`对象和`Object`对象的实例。最后那一行的空对象外面，之所以要加括号，是因为如果不加，JavaScript引擎会把一对大括号解释为一个代码块，而不是一个对象，从而导致这一行代码被解释为`{}; instanceof Object`，引擎就会报错。

注意，`instanceof`运算符只能用于对象，不适合用于原始类型的值。

```javascript
var s = 'hello';
s instanceof String // false

var s = new String('hello');
s instanceof String // true
```

上面代码中，字符串不是`String`对象的实例（因为字符串不是对象），所以返回`false`，而字符串对象是`String`对象的实例，所以返回`true`。

此外，`undefined`和`null`不是对象，所以`instanceOf`运算符总是返回`false`。

```javascript
undefined instanceof Object // false
null instanceof Object // false
```

如果存在继承关系，也就是说，`a`是`A`的实例，而`A`继承了`B`，那么`instanceof`运算符对`A`和`B`都返回`true`。

```javascript
var a = [];

a instanceof Array // true
a instanceof Object // true
```

上面代码表示，`a`是一个数组，所以它是`Array`的实例；同时，`Array`继承了`Object`，所以`a`也是`Object`的实例。

利用`instanceof`运算符，还可以巧妙地解决，调用构造函数时，忘了加`new`命令的问题。

```javascript
function Fubar (foo, bar) {
  if (this instanceof Fubar) {
    this._foo = foo;
    this._bar = bar;
  }
  else {
    return new Fubar(foo, bar);
  }
}
```

上面代码使用`instanceof`运算符，在函数体内部判断`this`关键字是否为构造函数`Fubar`的实例。如果不是，就表明忘了加`new`命令。

## this关键字

### 涵义

理解`this`关键字的含义，对于JavaScript开发的重要性怎么强调都不过分。

简单说，`this`就是指属性或方法“当前”所在的对象。

```javascript
this['property']
this.property
```

上面代码中，`this`就代表`property`属性当前所在的对象。

下面是一个实际的例子。

```javascript
var person = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

person.describe()
// "姓名：张三"
```

上面代码中，`this.name`表示`describe`方法所在的当前对象的`name`属性。调用`person.describe`方法时，`describe`方法所在的当前对象是`person`，所以就是调用`person.name`。

由于对象的属性可以赋给另一个对象，所以属性所在的当前对象是可变的，即`this`的指向是可变的。

```javascript
var A = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

var B = {
  name: '李四'
};

B.describe = A.describe;
B.describe()
// "姓名：李四"
```

上面代码中，`A.describe`属性被赋给`B`，于是`B.describe`就表示`describe`方法所在的当前对象是`B`，所以`this.name`就指向`B.name`。

稍稍重构这个例子，`this`的动态指向就能看得更清楚。

```javascript
function f() {
  return '姓名：'+ this.name;
}

var A = {
  name: '张三',
  describe: f
};

var B = {
  name: '李四',
  describe: f
};

A.describe() // "姓名：张三"
B.describe() // "姓名：李四"
```

上面代码中，函数`f`内部使用了`this`关键字，随着`f`所在的对象不同，`this`的指向也不同。

只要函数被赋给另一个变量，`this`的指向就会变。

```javascript
var A = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

var name = '李四';
var f = A.describe;
f() // "姓名：李四"
```

上面代码中，`A.describe`被赋值给变量`f`，内部的`this`就会指向`f`运行时所在的对象（本例是顶层对象）。

再看一个网页编程的例子。

```html
<input type="text" name="age" size=3 onChange="validate(this, 18, 99);">

<script>
function validate(obj, lowval, hival){
  if ((obj.value < lowval) || (obj.value > hival))
    console.log('Invalid Value!');
}
</script>
```

上面代码是一个文本输入框，每当用户输入一个值，就会调用`onChange`回调函数，验证这个值是否在指定范围。回调函数传入`this`，就代表传入当前对象（即文本框），然后就可以从`this.value`上面读到用户的输入值。

总结一下，JavaScript语言之中，一切皆对象，运行环境也是对象，所以函数都是在某个对象之中运行，`this`就是这个对象（环境）。这本来并不会让用户糊涂，但是JavaScript支持运行环境动态切换，也就是说，`this`的指向是动态的，没有办法事先确定到底指向哪个对象，这才是最让初学者感到困惑的地方。

如果一个函数在全局环境中运行，那么`this`就是指顶层对象（浏览器中为`window`对象）。

```javascript
function f() {
  return this;
}

f() === window // true
```

上面代码中，函数`f`在全局环境运行，它内部的`this`就指向顶层对象`window`。

可以近似地认为，`this`是所有函数运行时的一个隐藏参数，决定了函数的运行环境。

### 使用场合

`this`的使用可以分成以下几个场合。

**（1）全局环境**

在全局环境使用`this`，它指的就是顶层对象`window`。

```javascript
this === window // true

function f() {
  console.log(this === window); // true
}
```

上面代码说明，不管是不是在函数内部，只要是在全局环境下运行，`this`就是指顶层对象`window`。

**（2）构造函数**

构造函数中的`this`，指的是实例对象。

```javascript
var Obj = function (p) {
  this.p = p;
};

Obj.prototype.m = function() {
  return this.p;
};
```

上面代码定义了一个构造函数`Obj`。由于`this`指向实例对象，所以在构造函数内部定义`this.p`，就相当于定义实例对象有一个`p`属性；然后`m`方法可以返回这个p属性。

```javascript
var o = new Obj('Hello World!');

o.p // "Hello World!"
o.m() // "Hello World!"
```

**（3）对象的方法**

当`a`对象的方法被赋予`b`对象，该方法中的this就从指向`a`对象变成了指向`b`对象。所以要特别小心，将某个对象的方法赋值给另一个对象，会改变`this`的指向，具体例子请看前面一节。

有时，某个方法位于多层对象的内部，这时如果为了简化书写，把该方法赋值给一个变量，往往会得到意料之外的结果。

```javascript
var a = {
  b: {
    m: function() {
      console.log(this.p);
    },
    p: 'Hello'
  }
};

var hello = a.b.m;
hello() // undefined
```

上面代码中，`m`是多层对象内部的一个方法。为求简便，将其赋值给`hello`变量，结果调用时，`this`指向了顶层对象。为了避免这个问题，可以只将`m`所在的对象赋值给`hello`，这样调用时，`this`的指向就不会变。

```javascript
var hello = a.b;
hello.m() // Hello
```

**（4）Node**

在Node中，`this`的指向又分成两种情况。全局环境中，`this`指向全局对象`global`；模块环境中，`this`指向module.exports。

```javascript
// 全局环境
this === global // true

// 模块环境
this === module.exports // true
```

### 使用注意点

**（1）避免多层this**

由于`this`的指向是不确定的，所以切勿在函数中包含多层的`this`。

```javascript
var o = {
  f1: function () {
    console.log(this);
    var f2 = function () {
      console.log(this);
    }();
  }
}

o.f1()
// Object
// Window
```

上面代码包含两层`this`，结果运行后，第一层指向该对象，第二层指向全局对象。实际执行的是下面的代码。

```javascript
var temp = function () {
  console.log(this);
};

var o = {
  f1: function () {
    console.log(this);
    var f2 = temp();
  }
}
```

一个解决方法是在第二层改用一个指向外层`this`的变量。

```javascript
var o = {
  f1: function() {
    console.log(this);
    var that = this;
    var f2 = function() {
      console.log(that);
    }();
  }
}

o.f1()
// Object
// Object
```

上面代码定义了变量`that`，固定指向外层的`this`，然后在内层使用`that`，就不会发生`this`指向的改变。

事实上，使用一个变量固定`this`的值，然后内层函数调用这个变量，是非常常见的做法，有大量应用，请务必掌握。

JavaScript 提供了严格模式，也可以硬性避免这种问题。在严格模式下，如果函数内部的`this`指向顶层对象，就会报错。

```javascript
var counter = {
  count: 0
};
counter.inc = function () {
  'use strict';
  this.count++
};
var f = counter.inc;
f()
// TypeError: Cannot read property 'count' of undefined
```

上面代码中，`inc`方法通过`'use strict'`声明采用严格模式，这时内部的`this`一旦指向顶层对象，就会报错。

**（2）避免数组处理方法中的this**

数组的`map`和`foreach`方法，允许提供一个函数作为参数。这个函数内部不应该使用`this`。

```javascript
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);
    });
  }
}

o.f()
// undefined a1
// undefined a2
```

上面代码中，`foreach`方法的回调函数中的`this`，其实是指向`window`对象，因此取不到`o.v`的值。原因跟上一段的多层`this`是一样的，就是内层的`this`不指向外部，而指向顶层对象。

解决这个问题的一种方法，是使用中间变量。

```javascript
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    var that = this;
    this.p.forEach(function (item) {
      console.log(that.v+' '+item);
    });
  }
}

o.f()
// hello a1
// hello a2
```

另一种方法是将`this`当作`foreach`方法的第二个参数，固定它的运行环境。

```javascript
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);
    }, this);
  }
}

o.f()
// hello a1
// hello a2
```

**（3）避免回调函数中的this**

回调函数中的`this`往往会改变指向，最好避免使用。

```javascript
var o = new Object();

o.f = function () {
  console.log(this === o);
}

o.f() // true
```

上面代码表示，如果调用`o`对象的f方法，其中的`this`就是指向`o`对象。

但是，如果将`f`方法指定给某个按钮的`click`事件，this的指向就变了。

```javascript
$('#button').on('click', o.f);
```

点击按钮以后，控制台会显示`false`。原因是此时`this`不再指向`o`对象，而是指向按钮的DOM对象，因为`f`方法是在按钮对象的环境中被调用的。这种细微的差别，很容易在编程中忽视，导致难以察觉的错误。

为了解决这个问题，可以采用下面的一些方法对`this`进行绑定，也就是使得`this`固定指向某个对象，减少不确定性。

## 固定this的方法

`this`的动态切换，固然为JavaScript创造了巨大的灵活性，但也使得编程变得困难和模糊。有时，需要把`this`固定下来，避免出现意想不到的情况。JavaScript提供了`call`、`apply`、`bind`这三个方法，来切换/固定`this`的指向。

### call方法

函数的call方法，可以指定该函数内部this的指向（即函数执行时所在的作用域），然后在所指定的作用域中，调用该函数。

```javascript
var o = {};

var f = function (){
  return this;
};

f() === this // true
f.call(o) === o // true
```

上面代码中，在全局环境运行函数f时，this指向全局环境；call方法可以改变this的指向，指定this指向对象o，然后在对象o的作用域中运行函数f。

再看一个例子。

```javascript
var n = 123;
var o = { n : 456 };

function a() {
  console.log(this.n);
}

a.call() // 123
a.call(null) // 123
a.call(undefined) // 123
a.call(window) // 123
a.call(o) // 456
```

上面代码中，a函数中的this关键字，如果指向全局对象，返回结果为123。如果使用call方法将this关键字指向o对象，返回结果为456。可以看到，如果call方法没有参数，或者参数为null或undefined，则等同于指向全局对象。

call方法的完整使用格式如下。

```javascript
func.call(thisValue, arg1, arg2, ...)
```

它的第一个参数就是this所要指向的那个对象，后面的参数则是函数调用时所需的参数。

```javascript
function add(a,b) {
  return a+b;
}

add.call(this,1,2) // 3
```

上面代码中，call方法指定函数add在当前环境（对象）中运行，并且参数为1和2，因此函数add运行后得到3。

call方法的一个应用是调用对象的原生方法。

```javascript
var obj = {};
obj.hasOwnProperty('toString') // false

obj.hasOwnProperty = function (){
  return true;
};
obj.hasOwnProperty('toString') // true

Object.prototype.hasOwnProperty.call(obj, 'toString') // false
```

上面代码中，hasOwnProperty是obj对象继承的方法，如果这个方法一旦被覆盖，就不会得到正确结果。call方法可以解决这个方法，它将hasOwnProperty方法的原始定义放到obj对象上执行，这样无论obj上有没有同名方法，都不会影响结果。

### apply方法

apply方法的作用与call方法类似，也是改变this指向，然后再调用该函数。唯一的区别就是，它接收一个数组作为函数执行时的参数，使用格式如下。

```javascript
func.apply(thisValue, [arg1, arg2, ...])
```

apply方法的第一个参数也是this所要指向的那个对象，如果设为null或undefined，则等同于指定全局对象。第二个参数则是一个数组，该数组的所有成员依次作为参数，传入原函数。原函数的参数，在call方法中必须一个个添加，但是在apply方法中，必须以数组形式添加。

请看下面的例子。

```javascript
function f(x,y){
  console.log(x+y);
}

f.call(null,1,1) // 2
f.apply(null,[1,1]) // 2
```

上面的f函数本来接受两个参数，使用apply方法以后，就变成可以接受一个数组作为参数。

利用这一点，可以做一些有趣的应用。

**（1）找出数组最大元素**

JavaScript不提供找出数组最大元素的函数。结合使用apply方法和Math.max方法，就可以返回数组的最大元素。

```javascript
var a = [10, 2, 4, 15, 9];

Math.max.apply(null, a)
// 15
```

**（2）将数组的空元素变为`undefined`**

通过`apply`方法，利用Array构造函数将数组的空元素变成undefined。

```javascript
Array.apply(null, ["a",,"b"])
// [ 'a', undefined, 'b' ]
```

空元素与`undefined`的差别在于，数组的`forEach`方法会跳过空元素，但是不会跳过`undefined`。因此，遍历内部元素的时候，会得到不同的结果。

```javascript
var a = ['a', , 'b'];

function print(i) {
  console.log(i);
}

a.forEach(print)
// a
// b

Array.apply(null, a).forEach(print)
// a
// undefined
// b
```

**（3）转换类似数组的对象**

另外，利用数组对象的`slice`方法，可以将一个类似数组的对象（比如`arguments`对象）转为真正的数组。

```javascript
Array.prototype.slice.apply({0:1,length:1})
// [1]

Array.prototype.slice.apply({0:1})
// []

Array.prototype.slice.apply({0:1,length:2})
// [1, undefined]

Array.prototype.slice.apply({length:1})
// [undefined]
```

上面代码的`apply`方法的参数都是对象，但是返回结果都是数组，这就起到了将对象转成数组的目的。从上面代码可以看到，这个方法起作用的前提是，被处理的对象必须有length属性，以及相对应的数字键。

**（4）绑定回调函数的对象**

上一节按钮点击事件的例子，可以改写成

```javascript
var o = new Object();

o.f = function ()v {
  console.log(this === o);
}

var f = function (){
  o.f.apply(o);
  // 或者 o.f.call(o);
};

$('#button').on('click', f);
```

点击按钮以后，控制台将会显示`true`。由于`apply`方法（或者`call`方法）不仅绑定函数执行时所在的对象，还会立即执行函数，因此不得不把绑定语句写在一个函数体内。更简洁的写法是采用下面介绍的`bind`方法。

### bind方法

`bind`方法用于将函数体内的`this`绑定到某个对象，然后返回一个新函数。

```javascript
var print = console.log;
print(2)
// TypeError: Illegal invocation
```

上面代码中，我们将`console`对象的`log`赋给变量`print`，然后调用`print`就报错了，因为这时`log`方法内部的`this`已经不指向`console`对象了。

`bind`方法可以解决这个问题，让`log`方法绑定`console`对象。

```javascript
var print = console.log.bind(console);
print(2)
// 2
```

上面代码中，`bind`方法将`log`方法内部的`this`绑定到`console`对象，这时就可以安全地将这个方法赋值给其他变量了。

下面是一个更清晰的例子。

```javascript
var counter = {
  count: 0,
  inc: function () {
    this.count++;
  }
};

counter.count // 0
counter.inc()
counter.count // 1
```

上面代码中，`counter.inc`内部的`this`，默认指向`counter`对象。如果将这个方法赋值给另一个变量，就会出错。

```javascript
var counter = {
  count: 0,
  inc: function () {
    this.count++;
  }
};

var func = counter.inc;
func();
counter.count // 0
count // NaN
```

上面代码中，函数`func`是在全局环境中运行的，这时`inc`内部的`this`指向顶层对象`window`，所以`counter.count`是不会变的，反而创建了一个全局变量`count`。因为`window.count`原来等于`undefined`，进行递增运算后`undefined++`就等于`NaN`。

为了解决这个问题，可以使用`this`方法，将`inc`内部的`this`绑定到`counter`对象。

```javascript
var func = counter.inc.bind(counter);
func();
counter.count // 1
```

上面代码中，`bind`方法将`inc`方法绑定到`counter`以后，再运行`func`就会得到正确结果。

`this`绑定到其他对象也是可以的。

```javascript
var obj = {
  count: 100
};
var func = counter.inc.bind(obj);
func();
obj.count // 101
```

上面代码中，`bind`方法将`inc`方法内部的`this`，绑定到`obj`对象。结果调用`func`函数以后，递增的就是`obj`内部的`count`属性。

`bind`比`call`方法和`apply`方法更进一步的是，除了绑定`this`以外，还可以绑定原函数的参数。

```javascript
var add = function (x, y) {
  return x * this.m + y * this.n;
}

var obj = {
  m: 2,
  n: 2
};

var newAdd = add.bind(obj, 5);

newAdd(5)
// 20
```

上面代码中，`bind`方法除了绑定`this`对象，还将`add`函数的第一个参数`x`绑定成`5`，然后返回一个新函数`newAdd`，这个函数只要再接受一个参数`y`就能运行了。

如果`bind`方法的第一个参数是`null`或`undefined`，等于将`this`绑定到全局对象，函数运行时`this`指向顶层对象（在浏览器中为`window`）。

```javascript
function add(x, y) {
  return x + y;
}

var plus5 = add.bind(null, 5);
plus5(10) // 15
```

上面代码中，函数`add`内部并没有`this`，使用`bind`方法的主要目的是绑定参数`x`，以后每次运行新函数`plus5`，就只需要提供另一个参数`y`就够了。而且因为`add`内部没有`this`，所以`bind`的第一个参数是`null`，不过这里如果是其他对象，也没有影响。

对于那些不支持`bind`方法的老式浏览器，可以自行定义`bind`方法。

```javascript
if(!('bind' in Function.prototype)){
  Function.prototype.bind = function(){
    var fn = this;
    var context = arguments[0];
    var args = Array.prototype.slice.call(arguments, 1);
    return function(){
      return fn.apply(context, args);
    }
  }
}
```

`bind`方法有一些使用注意点。

**（1）每一次返回一个新函数**

`bind`方法每运行一次，就返回一个新函数，这会产生一些问题。比如，监听事件的时候，不能写成下面这样。

```javascript
element.addEventListener('click', o.m.bind(o));
```

上面代码中，`click`事件绑定`bind`方法生成的一个匿名函数。这样会导致无法取消绑定，所以，下面的代码是无效的。

```javascript
element.removeEventListener('click', o.m.bind(o));
```

正确的方法是写成下面这样：

```javascript
var listener = o.m.bind(o);
element.addEventListener('click', listener);
//  ...
element.removeEventListener('click', listener);
```

**（2）结合回调函数使用**

回调函数是JavaScript最常用的模式之一，但是一个常见的错误是，将包含`this`的方法直接当作回调函数。

```javascript
var counter = {
  count: 0,
  inc: function () {
    'use strict';
    this.count++;
  }
};

function callIt(callback) {
  callback();
}

callIt(counter.inc)
// TypeError: Cannot read property 'count' of undefined
```

上面代码中，`counter.inc`方法被当作回调函数，传入了`callIt`，调用时其内部的`this`指向`callIt`运行时所在的对象，即顶层对象`window`，所以得不到预想结果。注意，上面的`counter.inc`方法内部使用了严格模式，在该模式下，`this`指向顶层对象时会报错，一般模式不会。

解决方法就是使用`bind`方法，将`counter.inc`绑定`counter`。

```javascript
callIt(counter.inc.bind(counter));
counter.count // 1
```

还有一种情况比较隐蔽，就是某些数组方法可以接受一个函数当作参数。这些函数内部的`this`指向，很可能也会出错。

```javascript
var obj = {
  name: '张三',
  times: [1, 2, 3],
  print: function () {
    this.times.forEach(function (n) {
      console.log(this.name);
    });
  }
};

obj.print()
// 没有任何输出
```

上面代码中，`obj.print`内部`this.times`的`this`是指向`obj`的，这个没有问题。但是，`forEach`方法的回调函数内部的`this.name`却是指向全局对象，导致没有办法取到值。稍微改动一下，就可以看得更清楚。

```javascript
obj.print = function () {
  this.times.forEach(function (n) {
    console.log(this === window);
  });
};

obj.print()
// true
// true
// true
```

解决这个问题，也是通过`bind`方法绑定`this`。

```javascript
obj.print = function () {
  this.times.forEach(function (n) {
    console.log(this.name);
  }.bind(this));
};

obj.print()
// 张三
// 张三
// 张三
```

**（3）结合`call`方法使用**

利用`bind`方法，可以改写一些JavaScript原生方法的使用形式，以数组的`slice`方法为例。

```javascript
[1, 2, 3].slice(0, 1)
// [1]

// 等同于

Array.prototype.slice.call([1, 2, 3], 0, 1)
// [1]
```

上面的代码中，数组的slice方法从`[1, 2, 3]`里面，按照指定位置和长度切分出另一个数组。这样做的本质是在`[1, 2, 3]`上面调用`Array.prototype.slice`方法，因此可以用`call`方法表达这个过程，得到同样的结果。

`call`方法实质上是调用`Function.prototype.call`方法，因此上面的表达式可以用`bind`方法改写。

```javascript
var slice = Function.prototype.call.bind(Array.prototype.slice);

slice([1, 2, 3], 0, 1) // [1]
```

可以看到，利用bind方法，将`[1, 2, 3].slice(0, 1)`变成了`slice([1, 2, 3], 0, 1)`的形式。这种形式的改变还可以用于其他数组方法。

```javascript
var push = Function.prototype.call.bind(Array.prototype.push);
var pop = Function.prototype.call.bind(Array.prototype.pop);

var a = [1 ,2 ,3];
push(a, 4)
a // [1, 2, 3, 4]

pop(a)
a // [1, 2, 3]
```

如果再进一步，将`Function.prototype.call`方法绑定到`Function.prototype.bind`对象，就意味着`bind`的调用形式也可以被改写。

```javascript
function f() {
  console.log(this.v);
}

var o = { v: 123 };

var bind = Function.prototype.call.bind(Function.prototype.bind);

bind(f, o)() // 123
```

上面代码表示，将`Function.prototype.call`方法绑定`Function.prototype.bind`以后，`bind`方法的使用形式从`f.bind(o)`，变成了`bind(f, o)`。

## 参考链接

- Jonathan Creamer, [Avoiding the "this" problem in JavaScript](http://tech.pro/tutorial/1192/avoiding-the-this-problem-in-javascript)
- Erik Kronberg, [Bind, Call and Apply in JavaScript](https://variadic.me/posts/2013-10-22-bind-call-and-apply-in-javascript.html)
- Axel Rauschmayer, [JavaScript’s this: how it works, where it can trip you up](http://www.2ality.com/2014/05/this.html)
