# JS 基础 - 对象

对象是 JavaScript 中常用的一种类型，可以存储键值对或者复杂的实体数据。

除了自定义的一些对象之外，ECMAScript Specification 中还有`Build in`内置对象，包括`Global Object`（全局对象）和`Fundamental Object`（基本对象），下面列举一下这些内置对象以及全局对象。

有关对象基本操作可以看[JS this 以及原型、原型链](/front/javascript/primary/JSthis以及原型.html#理解对象)这篇文章。

上图基本上已经枚举了 JavaScript 中所有的内置对象，不过大部分在一般的开发中是用不到的，常用到子类型有以下几种:

* Object
* Array
* Function
* String
* Number
* Boolean
* Date
* Error
* RegExg

### object 内容

object 的内容无非有属性（可计算属性）、方法、对象的复制、属性描述符（数据属性和访问属性）、对象遍历等

#### 可计算属性名

可计算属性名是**es6**新增的特性

```javascript
let obj = {
  a: 1,
  b: 2,
  // 可计算属性名
  ["f" + "oo"]() {
    console.log("foo");
  },
};

obj.bc; // 3
obj["bc"]; // 3
obj.foo(); // foo
```

#### 属性描述器

属性描述器是从**ES5**开始才有的，可以看到对象中某个属性的特性，比如`writable`\(可访问性\)、`configurable`（可配置性）、`enumerable`（可枚举性）以及 `value`\(值\)

获取对象的某个特性 _Object_ 也提供了方法 _Object.getOwnPropertyDescriptor\(obj, 'a'\)_

同时也有定义对象的某个特性的方式 _Object.defineProperty_

```javascript
// Object.getOwnPropertyDescriptor

let obj = { a: 1, b: 2 };
Object.getOwnPropertyDescriptor(obj, "a"); // 获取属性 a 的描述特定

Object.defineProperty(obj, "c", {
  writable: true,
  configurable: true,
  enumerable: true,
  value: 3,
});
obj; // {a: 1, b: 2, c:3}
```

#### 不变性

对象的不变性其实就是对对象的属性描述器特性进行修改，有以下几种方式：

* Object.preventExtensions - 禁止一个对象添加新属性保留原来的属性
* Object.seal - 在现有的对象调用 _preventExtensions_，并对所有属性的_configurable_ 标记为_false_
* Object.freeze - 在现有的对象上调用_seal_，并对所有属性的_writable_ 标记为_false_

#### 属性访问器

属性访问器也就在访问对象属性所调用的，也就是常说的**setter**和**getter**，其语言内部调用 **\[\[Get\]\]** 和 **\[\[Put\]\]** 如果要对属性的值有自定义的操作（数据劫持），可以使用使用他们。属性访问器关心的是 **configurable** 和 **enumerable**

```javascript
let obj = { a: "" };

// 不能配置 writable 否则 get set 不生效
Object.defineProperty(obj, "a", {
  enumerable: true,
  configurable: true,
  get a() {
    return "3";
  },
  set a(val) {
    this.a = val;
  },
});
```

### 

### 

### 

### 

### 

### 

