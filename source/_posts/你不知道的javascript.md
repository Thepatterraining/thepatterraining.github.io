---
title: 你不知道的javascript
date: 2019-06-03 13:34:52
tags: [node, 你不知道的javascript, js]
category: js
article: 你不知道的javascript上
---

# 你不知道的 javascript

## 作用域

作用域了解了 lhs 和 rhs 两种查找方式

- lhs, 赋值左侧查找，需要找到变量，然后进行赋值，如果没找到，在非严格模式下会创建变量，严格模式会抛出 ReferenceError 异常
- rhs，非赋值左侧查找，在引用的时候使用这种查找，在作用域里面如果没有查到，会直接抛出 ReferenceError 异常，而如果你查到了，但是操作不合法，比如试图对一个非函数类型的值进行函数调用，或者引用 null 或 undefined 类型的值中的属性，那么会抛出 TypeError 异常

`ReferenceError`和作用域的判别失败有关，而`TypeError`则是代表作用域里面找到了，但是对结果的操作是非法或者不合理的

<!--more-->

### 动态作用域

动态作用域和词法作用域的区别，词法作用域是在写代码或者说定义时候确定的，而动态作用域是在运行时确定的。（this 也是），词法作用域关注函数在何处声明，而动态作用域关注函数从何处调用。

## 闭包

当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行

比如

```javascript
function foo() {
  var a = 2;
  function bar() {
    console.log(a);
  }

  bar();
}

foo();
```

基于词法作用域的规则，函数 bar 可以访问外部的变量 a(这个例子中的 a 是一个 rhs 查询)

上面的代码从技术来讲，也许是闭包，但根据前面的定义，确切的说并不是。

下面的代码清晰的展示了闭包

```javascript
function foo() {
  var a = 2;

  function bar() {
    console.log(a);
  }

  return bar;
}

var baz = foo();

baz(); //2           这就是闭包的效果
```

函数 bar()的词法作用域能够访问 foo()的内部作用域。然后我们将 bar()函数本身当作一个值类型进行传递。在这个例子中，我们将 bar 所引用的函数本身当作返回值

在 foo()执行后，其返回值（也就是内部的 bar()函数）赋值给变量 baz 并调用 baz()，实际上只是通过不同的标识符引用调用了内部的函数 bar()

bar()显然可以被正常执行，但是在这个例子中，他在自己定义的词法作用域以外的地方执行。

在 foo()执行后，通常会期待 foo()的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器来释放不再使用的内存空间。由于看上去 foo()的内容不会再被使用，所以很自然的会考虑对其进行回收。

拜 bar()所声明的位置所赐，它拥有涵盖 foo()内部作用域的闭包，使得该作用域能够一直存活，以供 bar()在之后任何时间进行引用

bar()依然持有对该作用域的引用，而这个引用就叫做闭包

    当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，这时就产生了闭包。

## this

### 绑定规则

#### 默认绑定

首先是最常用的函数调用类型：独立函数调用。可以把这条规则看作是无法应用其他规则时的默认规则

思考一下下面的代码

```javascript
function foo() {
  console.log(this.a);
}

var a = 2;
foo();
```

你应该注意到的，当我们调用 foo()时，this.a 被解析成了全局变量 a。为什么？因为在本例中，函数调用时应用了 this 的默认绑定，因此 this 指向全局对象。

在代码中，foo()是直接使用不带任何修饰的函数引用进行调用的，因此只能使用默认绑定，无法应用其他规则

如果使用严格模式，那么全局对象将无法使用默认绑定，因此 this 会绑定到 undefined

```javascript
function foo() {
  "use strict";
  console.log(this.a);
}

var a = 2;
foo(); //typeError: this is undefined
```

#### 隐式绑定

另一条需要考虑的规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含，不过这种说法可能会造成一些误导

思考下面代码

```javascript
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2,
  foo: foo
};
obj.foo(); //2
```

这个代码里面，使用 obj.foo()来调用，调用位置会使用 obj 上下文来引用函数，因此你可以说函数被调用时 obj 对象“拥有”或者“包含”它

无论你如何称呼这个模式，当 foo()被调用时，它的落脚点确实指向 obj 对象。当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。因为调用 foo()时，this 被绑定到 obj，因此 this.a 和 obj.a 时一样的

对象属性引用链中只有最顶层或者说最后一层会影响调用位置。举例来说：

```javascript
function foo() {
  console.log(this.a);
}

var obj2 = {
  a: 42,
  foo: foo
};

var obj1 = {
  a: 2,
  obj2: obj2
};

obj1.obj2.foo(); //42
```

###### 隐式丢失

一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把 this 绑定到全局对象或者 undefined 上，取决于是否是严格模式

思考下面的代码

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
};
var bar = obj.foo; //函数别名

var a = "oops, global"; //a是全局对象的属性

bar(); //oops global
```

一种更微妙，更常见并且更出乎意料的情况发生在传入回调函数时：

```javascript
function foo() {
  console.log(this.a);
}

function doFoo(fn) {
  //fn其实引用的是foo
  fn(); //调用位置
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global"; //a是全局对象的属性
doFoo(obj.foo); //oops, global
```

参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值，所以结果和上一个例子一样

如果把函数传入语言内置的函数而不是你自己声明的函数，会发生什么呢？结果是一样的，没有区别：

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global"; //a是全局对象的属性
setTimeout(obj.foo, 100); //oops, global
```

js 内置的 setTimeout 函数实现和下面的伪代码类似：

```javascript
function setTimeout(fn, delay) {
  //等待delay毫秒
  fn(); //调用位置
}
```

就像我们看到的，回调函数丢失 this 绑定是非常常见的。无论哪种情况，this 的改变都是意想不到的，实际上你无法控制回调函数的执行方式，因此就没有办法控制会影响绑定的调用位置，之后我们会介绍如何通过固定 this 来修复这个问题

#### 显式绑定

js 中的“所有”函数都有一些有用的特性，可以用来显示绑定，比如 call 和 apply 方法。严格来说，js 的宿主环境有时会提供一些非常特殊的函数，它们并没有这两个方法。但是这样的函数非常罕见。

可惜，显示绑定仍然无法解决我们之前提出的丢失绑定问题

但是显示绑定的一个变种可以解决这个问题

##### 硬绑定

思考下面的代码

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2
};

var bar = function() {
  foo.call(obj);
};

bar(); //2
setTimeout(bar, 100); //2
//硬绑定的bar不可能再修改它的this
bar.call(window); //2
```

由于硬绑定是一种非常常用的模式，所以在 es5 中内置了 Function.prototype.bind 方法，bind 返回一个硬编码的新韩淑，他会把参数设置 this 的上下文并调用原始函数

#### new 绑定

这是最后一条 this 的绑定规则，在讲解它之前我们首先需要澄清一个非常常见的关于 js 中函数和对象的误解

在传统的面向类的语言中，“构造函数”是类中的一些特殊方法，使用 new 初始化类时会调用类中的构造函数。通常的形式是这样的：

    something = new MyClass()

js 也有一个 new 操作符，使用方法看起来一样，然而，js 中 new 的机制实际上和面向类的语言完全不同

js 中的构造函数只是一些使用 new 操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，他们甚至都不能说时一种特殊的构造函数，他们只是被 new 的普通函数而已。

这里有一个重要但是非常细微的区别：实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作

1. 创建一个全新的对象
2. 这个新对象会被执行【【原型】】连接
3. 这个新对象会绑定到函数调用的 this
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象

### 优先级

这些规则如果同时出现，就需要一个优先级类进行判断是用的哪一条规则

毫无疑问，默认绑定的优先级是最低的，所以我们可以先不考虑它

隐式和显示哪个优先级更高呢，我们来测试一下

```javascript
function foo() {
  console.log(this.a);
}

var obj1 = {
  a: 2,
  foo: foo
};

var obj2 = {
  a: 3,
  foo: foo
};

obj1.foo(); //2
obj2.foo(); //3

obj1.foo.call(obj2); //3
obj2.foo.call(obj1); //2
```

可以看到，显示的优先级更高，也就是说在判断时应当先考虑是否可以应用显示绑定

现在我们需要搞清楚 new 绑定和隐式绑定的优先级

```javascript
function foo(something) {
  this.a = something;
}

var obj1 = {
  foo: foo
};

var obj2 = {};

obj1.foo(2);
console.log(obj1.a); //2

obj1.foo.call(obj2, 3);
console.log(obj2.a); //3

var bar = new obj1.foo(4);
console.log(obj1.a); //2
console.log(bar.a); //4
```

可以看到 new 绑定比隐式绑定优先级高。但是 new 绑定和显示绑定谁的优先级更高呢

我们可以用硬绑定来试一下

```javascript
function foo(something) {
  this.a = something;
}

var obj1 = {};

var bar = foo.bind(obj1);

bar(2);
console.log(obj1.a); //2

var baz = new bar(3);
console.log(obj1.a); //2
console.log(baz.a); //3
```

判断 this

1. 如果用 new，那么 this 绑定的是新创建的对象
2. 如果通过 call,apply 或者 bind，this 绑定的是指定的对象
3. 函数是否在某个上下文对象中调用，如果是的话，绑定的是上下文对象，比如 obj.foo()
4. 如果都不是，那么默认绑定到全局对象，如果严格模式，就绑定到 undefined

### 绑定例外

规则总有例外，这里也一样

如果你把 null 或者 undefined 作为 this 的绑定对象传入 call,apply 或者 bind，这些值在调用时会被忽略，实际应用的是默认绑定规则

```javascript
function foo() {
  console.log(this.a);
}
var a = 2;
foo.call(null); //2
```

### 箭头函数

箭头函数不使用 this 的四种标准规则，而是根据外层（函数或者全局）作用域来决定 this

```javascript
function foo() {
  return a => {
    //this 继承自foo()
    console.log(this.a);
  };
}

var obj1 = {
  a: 2
};

var obj2 = {
  a: 3
};

var bar = foo.call(obj1);

bar.call(obj2); //2 不是 3
```

foo 内部的箭头函数会捕获调用时 foo 的 this，由于 foo 的 this 绑定到 obj1,bar 的 this 也会绑定到 obj1，箭头函数的绑定无法被修改

箭头函数最常用于回调函数

```javascript
function foo() {
  setTimeout(() => {
    //这里的this继承自foo
    console.log(this.a);
  }, 100);
}

var obj = {
  a: 2
};

foo.call(obj); //2
```

箭头函数的重要性体现在它用更常见的词法作用域取代了传统的 this 机制。

## 继承

js 一开始就没有设计成面向类的语言，所以也没有`class`, `extends`这种继承机制，但是`class`是一个一切皆对象的语言。
class 需要`new`来实例化一个对象，而`new`的过程中会执行构造函数，所以 js 的`new`操作符，就是直接把`普通函数`当成`构造函数`执行，用这样的方式来实现实例化，那么怎么实现`继承`机制呢，就引入了`prototype`，prototype 指向一个新的对象，这个对象里面存放了可以`共享`的属性。我们一般把 prototype 指向的这个属性叫做`原型对象`。

看下面的代码

```javascript
function Foo() {}

var obj = new Foo();
```

在上面的代码中，obj 有一个**proto**属性指向 Foo.prototype 这个对象，这就是原型链，**proto**这个属性可以一直往上查找，直到原型链的顶层 Object.prototype 这里，这也就是在 js 里面可以用 map,foreach 这些方法的原因，他们本身是没有这些方法的，所以 js 会通过原型链进行查找，直到找到这个方法进行调用，如果没有找到，则会抛出 TypeError 异常，比如在上面的 obj 对象中调用 obj.log 方法，就会报错了。

### class

在 es6 中，引入了 class 语法，让 js 用起来像其他的面向对象一样，但其实不一样的，class 只是一个语法糖，他本身还是使用的 prototype 来实现的。

### 对象委托

js 里面完全可以使用一种对象委托的方式来实现继承，而不是类的形式，比如用类来完成一件事的话大概像下面一样

```javascript
function Foo(msg) {
  this.msg = msg;
}

Foo.prototype.identify = function() {
  return `I am ${this.msg}`;
};

function Bar(msg) {
  Foo.call(this, msg);
}

Bar.prototype = Object.create(Foo.prototype);

Bar.prototype.speak = function() {
  console.log(`Hello, ${this.identify()}`);
};

var b1 = new Bar("b1");
var b2 = new Bar("b2");

b1.speak();
b2.speak();
```

子类 Bar 继承了父类 Foo，然后生成了 b1,b2 两个实例

下面我们看看使用对象委托的方式来写同样的代码

```javascript
Foo = {
  init(msg) {
    this.msg = msg;
  },
  identify() {
    return `I am ${this.msg}`;
  }
};

Bar = Object.create(Foo);

Bar.speak = function() {
  console.log(`Hello, ${this.identify()}`);
};

var b1 = Object.create(Bar);
b1.init("b1");
var b2 = Object.create(Bar);
b2.init("b2");

b1.speak();
b2.speak();
```

这段代码看起来是不是简洁了呢，我们只是把对象关联了起来，而不用再去模仿类的行为。

当然了，es6 引入的 class 语法也会让你觉得简洁

```javascript
class Foo {
  constructor(msg) {
    this.msg = msg;
  }
  identify() {
    return `I am ${this.msg}`;
  }
}

class Bar extends Foo {
  constructor(msg) {
    super(msg);
  }

  identify() {
    return "I am";
  }

  speak() {
    console.log(`Hello, ${this.identify()}`);
  }
}

var b1 = new Bar("b1");
var b2 = new Bar("b2");

b1.speak();
b2.speak();
```

但是 class 只是把内部实现隐藏了起来，他的本质依旧是我们上面写的那样，依旧是使用的 prototype。

### 小结

我觉得 class 和对象委托这两种设计模式，他们的本质都是使用 prototype，只不过一种是在模仿类，而一种则是不用模仿类，我遇到这个请求的时候，我就把请求委托给另外一个对象。
