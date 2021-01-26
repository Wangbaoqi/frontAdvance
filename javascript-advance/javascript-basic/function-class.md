# JS 基础 - 函数、类

> function 相当于一个'子程序'，由外部代码或者自身来调用。在 JavaScript 中，function 是一流的对象，可以像对象那样具有属性和方法。

### Function 函数

在 JavaScript 中，每个 function 都是一个`Function object`，都会继承其属性和方法。

```javascript
(function() {}.constructor === Function); // true
```

调用`Function`构造函数，可以创建 function 函数，尽管这种创建跟`eval`类似，但是通过这种方式创建的是可以在全局范围里调用的。

_实例属性_

1. Function.arguments

   与传递函数的参数对应的数组，可以在函数中调用`arguments`使用，不推荐将其作为参数传递。

2. Function.length

   指定函数期望的参数数目

3. Function.name

   函数的名称

_实例方法_

1. Function.prototype.apply\(thisArg, argArray\)
2. Function.prototype.call\(thisArg, ...arg\)
3. Function.prototype.bind\(thisArg, ...arg\)
4. Function.prototype.toString\(\)

`function`可以接受传入参数，也可以返回值。如果在函数体内更改非引用类型的值时，外部对应的值时不会改变的，如果是引用类型的值，则是改变的、

#### 定义函数

函数的定义通常有以下几种方式

1. 函数声明

函数声明也是常用的一种定义函数的方法

```javascript
function foo(params) {
  // statement
}
```

1. 函数表达式

函数表达式类似于函数声明，函数表达式可以为函数定义名称，也可以是匿名的。使用函数表达式不能在其之前使用它，因为它不会被挂起在作用域的开头。 创建命名函数表达式的好处是在追踪报错时，可以在堆栈信息中查到函数名称。

```javascript
var foo = function() {};
// 命名函数表达式
var foos = function bar() {};
// arrow function
var fooa = () => {};
```

1. IIFE（立即执行函数表达式）

当函数只需要执行一次时，就可以使用 IIFE 了，也可以用来解决作用域问题。

```javascript
(function foo() {
  // statement
})();
```

除此之外，定义函数可以在运行时中字符串中定义，类似于`eval`

### Class 类

**Class** 的出现会让 JS 的面向对象编程更像传统语言，是一种语法糖，让面向对象的实现更加清晰一点。

#### constructor

**constructor**构造函数是类默认的方法，可以通过`new`命令生成实例时，自动调用此方法，如果没有显示定义该方法，空的**constructor**会被默认添加。

```javascript
class Foo {
  constructor() {}
}
let foo = new Foo();
Foo.prototype === foo.__proto__; //true
```

#### 类的实例

类的实例通过调用`new`关键字产生的，实例上的属性除非是**显式**在构造函数中定义的，否则实例是获取不到的。只能从实例的原型中获取。

```javascript
class Foo {
  _bar = "bar"; // 实例属性新写法
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  toString() {
    return `x-${this.x},y-${this.y}`;
  }
  getValue() {
    return this.x;
  }
}
let foo = new Foo(3, 4);
foo.getValue(); // 3 调用原型中的方法
foo; // {x: 3, y: 4}
foo.__proto__; // { construtor: class Foo, toString: f, getValue: f}
foo.hasOwnProperty("x");
foo.hasOwnProperty("toString");
foo.hasOwnProperty("getValue");
```

#### this 的指向

一般来讲，this 的指向是实例，但是有时结果就不一样了。

```javascript
class Foo {
  constructor() {
    // bind 方式绑定
    this.getBar = this.getBar.bind(this);
  }
  getBar() {
    return this.getRes();
  }

  getRes() {
    return "result";
  }
}
let foo = new Foo();
foo.getBar(); // 'result' 显式调用，this就是 当前实例 foo

const { getBar } = new Foo();
getBar(); // TypeError Cannot read property 'getRes' of undefined
```

后者显然是找不到当前的 this 了，因此可以在声明的时候，可以给对应的方法绑定**this**。React 中是不是也是这种方式呢？

```javascript
class Foo {
  constructor() {
    // bind 方式绑定
    this.getBar = this.getBar.bind(this);
  }
  getBar() {
    return this.getRes();
  }

  getRes() {
    return "result";
  }
}

// Arrow 函数 绑定this
class Foo {
  // class Field
  getBar = () => {
    return this.getRes();
  };
  getRes() {
    return "result";
  }
}

const { getBar } = new Foo();
getBar(); // result
```

#### 静态方法

类的定义中，定义的方法都会被实例继承，而如果在方法前面加上`static`，则这个方法只有类本身可以访问。 静态方法中的**this**指向是当前类。静态方法可以通过**extends**来继承，静态方法也可以从**super**上调用的。

```javascript
class Foo {
  static bar() {
    return this.baz();
  }
  baz() {
    return "baz";
  }
  static baz() {
    return "static baz";
  }
}

class Bar extends Foo {
  constructor(props) {
    super(props);
    this.barProps = props;
  }
  static getBar() {
    return super.bar();
  }
}
Foo.bar(); //
let foo = new Foo();
let bar = new Bar();
```

#### 静态属性

类的定义中，有了静态方法，那静态属性是怎么定义的，在**ES6**中明确确定，是没有静态属性的，不过在_提案_中，有了一种新方式，也是用`static`

```javascript
class Foo {
  fooName = "foo";
  // 新 静态属性
  static fooAge = 30;

  constructor() {
    this.bav = "bav";
  }
  bac() {}
  static bar() {
    return "bar";
  }
}

// 旧 静态属性
Foo.fooAge; // foo
```

