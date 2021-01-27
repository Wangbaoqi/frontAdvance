# JS 基础 - 作用域以及闭包

> 我们将“作用域”定义为一套规则，这套规则用来管理引擎如何在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找

### 理解作用域

* 编译原理
* 作用域嵌套
* 异常

#### 编译原理

> 代码执行之前会被编译，传统的编译方式和JavaScript引擎的编译方式类似，甚至JavaScript的编译步骤更加复杂一些

传统的编译步骤：

* 分词/词法分析
* 解释/语法分析 - 性能运行优化的步骤
* 代码生成 - 冗余元素的优化

解析： 代码在编译的过程中，会有这两个步骤： 

1. 编译器首先会创建一个新的变量\(如果当前作用域中没有同名的变量\) 

2. 然后编译器会为引擎生成执行时的代码，例如 a = 2 这个赋值操作，引擎首先会向当前作用域集合中查找是否有a这个变量，有的话，直接使用，没有的话，继续作用域链中查找；在这个过程中，编译器生成了代码，引擎在作用域中怎么查找变量a的呢？ 这时，引擎会为变量a执行LHS查询，当然还有一个RHS查询。其两者的含义是赋值操作的左侧和右侧。通俗的来说，对查找的目标是赋值的操作，就是LHS查询；如果要获取变量的值，则是RHS查询。

```javascript
function fn(a) {
  var b = a;
  return b;
}
fn(2);
// LHS有：(a , b= , 
// RHS有：fn( , a , b ,
```

#### 作用域嵌套

如果一个函数中包含了另一个函数，此时就会有作用域嵌套；如果查找一个变量，在当前的作用域中没有找到，就会向上一个作用域查找，直到查到作用域顶部，也就是全局作用域。如果全局中也没有找到，就会抛出异常

#### 异常

如果一个变量没有声明，引擎在作用域使用LHS和RHS查找锁抛出的异常是不同的。RHS会抛出来 referenceType异常。 而使用LHS在非严格模式下，会隐式的创建全局变量；如果使用严格模式，也会抛出referenceType异常。此外，如果使用RHS查找，并且对其使用非合法的操作时，会抛出TypeError异常

#### 词法作用域 - 欺骗词法

**eval**

JavaScript 中的 eval\(..\) 函数可以接受一个字符串为参数，并将其中的内容视为好像在书 写时就存在于程序中这个位置的代码。换句话说，可以在你写的代码中用程序生成代码并 运行，就好像代码是写在那个位置的一样。

```javascript
function foo(str, a) {
  eval(str);
  console.log(a, b);
}
var b = 3;
foo('var b = 2', 3); // 3, 2
```

**with**

with 通常被当作重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象 本身。\(并且现在也不推荐使用\)

```javascript
var foo = {
  a: 1,
  b: 2,
  c: 3
}
// 
foo.a = 5;
foo.b = 6;
foo.c = 7;

// 使用with
with(foo) {
  a = 5,
  b = 6;
  c = 7
}
```

在看with的另一个🌰：

```javascript
function foo(obj) {
  with(obj){
    a = 3;
  } 
}
var fo = {
  a = 4;
}
var bo = {
  b = 2;
}
foo(fo);
console.log(fo.a) // 3
foo(bo);
console.log(bo.b) // 2
console.log(bo.a)  // undefined
console.log(a) // 3
```

console.log\(a\) 为什么会是3？ ---- with 可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对 象的属性也会被处理为定义在这个作用域中的词法标识符。

性能问题：如果代码中大量使用 eval\(..\) 或 with，那么运行起来一定会变得非常慢。无论引擎多聪 明，试图将这些悲观情况的副作用限制在最小范围内，也无法避免如果没有这些优化，代 码会运行得更慢这个事实。

### 作用域链

> 作用域链，则需要结合函数的嵌套来说明。当定义一个函数时，它实际上创建了一个作用域节点，该节点上存储着当前作用域中的局部变量，并且该节点会挂载在作用域链的底端。在该函数中嵌套定义另一个函数时，同样会创建另一个函数作用域的节点，该节点同样也存储着当前函数作用域中的局部变量，在作用域链中会将该节点挂载在外层函数的节点之下。所以在进行变量访问时，会从自身节点开始查找，如果未找到变量的对应值，则会继续查找上一个节点。而由这一系列节点所串联起来的链就是我们所说的作用域链。

JavaScript中的函数采用静态作用域，也称词法作用域。当在执行函数调用时，不管何时何地执行函数，其中的变量在函数定义时就已经决定了，函数会从自身作用域节点开始，沿着作用域链向上访问变量的值。

### 作用域闭包

当函数可以记住并访问所在词法作用域时，就产生了闭包；即函数在当前词法作用域外执行。 另一种解释：闭包就是有权访问另一个函数作用域中的变量的函数

#### 静态作用域和动态作用域

JS采用的静态作用域，在函数定义的时候就将作用域决定了，因此，作用域链也在创建函数的时候创建的。 所谓动态作用域的话，是动态调用的。 下面看两段代码：

```javascript
var value = 2;
function foo() {
  console.log(value);
}
function bar() {
  var value = 1;
  foo();
}
bar() // 2
```

正是JS采用了静态作用域 执行的时候如果当前作用域中找不到，会在书写的位置找

再看个例子-来自JavaScript权威指南

```javascript
var scope = 'globalScope';
function checKScope1() {
  var scope = 'innerScope';
  function scope(){
    console.log(scope);
    return scope;
  }
  return scope()
}
checkScope1();
```

```javascript
var scope = 'globalScope';
function checKScope2() {
  var scope = 'innerScope';
  function scope(){
    console.log(scope);
    return scope;
  }
  return scope
}
checkScope2()();
```

依据静态作用域，结果显而易见，都是innerScope，不管是什么调用方式，在函数创建的时候，其作用域链都已经创建完成

就拿上述两个函数checkScope1 和 checkScope2 来讲，严格意义上checkScope2函数就产生了闭包，当前嵌套函数scope在当前词法作用域之外执行

使用执行上下文的概念来用图解释下作用域链

![&#x8BCD;&#x6CD5;&#x4F5C;&#x7528;&#x57DF;](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/JavaScript/ctx/scope_1.png)

```javascript
var fn;

function foo() {
  var a = 2;
  function bar() {
    console.log(a)
  }
  fn = bar;
}

function baz(fn) {
  fn(); // 产生了闭包
}
foo()
baz()
```

**无论通过何种手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包**

#### 实质问题

为了彻底搞懂闭包，先看一个例子：

```javascript
function wait(mes) {
  setTimeout(function timer() {
    console.log(mes)
  }, 1000)
}
wait('closure')
```

将一个内部函数\(名为 timer\)传递给 setTimeout\(..\)。timer 具有涵盖 wait\(..\) 作用域的闭包，因此还保有对变量 mes 的引用。

本质上无论何时何地，如果将函数\(访问它们各自的词法作用域\)当作第一 级的值类型并到处传递，你就会看到闭包在这些函数中的应用。在定时器、事件监听器、 Ajax 请求、跨窗口通信、Web Workers 或者任何其他的异步\(或者同步\)任务中，只要使 用了回调函数，实际上就是在使用闭包!

#### 循环和闭包

说明闭包，看一下常见的例子 for循环

```javascript
for(var i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i)
  }, i*1000)
}
```

结果很明显，每隔1s输出5，共5次

**解析:** setTimeout 是一个独立的线程，当主线程执行完成之后，在这里就是for循环执行完成之后，事件队列中的事件才会逐步执行，因此，setTimeout的执行是在for循环结束之后才会执行， 而5个定时器的回调函数timer中引用的i是共享的，此时是5；所以要按预期输出的话，必须把每个回调函数的作用域封闭起来\(也就是每次迭代需要一个块级作用域\)

1. 使用IIFE 立即执行函数 

```javascript
for(var i = 1; i < 5; i++) {
  (function(i) {
    setTimeout(function() {
      console.log(i)
    }, i*1000)
  })(i)
}
```

   2. 块级作用域

```javascript
for(let i = 1; i < 5; i++) {
  setTimeout(function() {
    console.log(i)
  }, i*1000)
}
```



