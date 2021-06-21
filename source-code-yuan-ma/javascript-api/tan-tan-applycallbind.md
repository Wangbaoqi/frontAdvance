# apply、call、bind实现

这几个函数的功能其实都是改变this的指向，前面在[「JavaScript基础 - this」](../../javascript-advance/javascript-basic/this.md#xian-shi-bang-ding)的时候简单的阐述过。接下来全面的学习一下这几个函数。💪

### apply和call

都知道apply和call的之间的区别，是传的参数不同，apply传的是数组参数，而call传的是列表参数，这两个函数是直接执行的

```javascript
let person = {
  title: 'nate'
}
function foo(name, age) {
  console.log(name)
  console.log(age)
  console.log(this.title)
  return name
}
foo.call(person, 'baoqi', 18) // 'baoqi' 18 nate
foo.apply(person, ['baoqi', 18]) // 'baoqi' 18 nate
```

结果是显而易见的，但是想知道这种方式是怎么执行的，是需要花费点时间研究的。下面手动实现以下apply和call

### 手动实现apply

```javascript
// call 手动实现
Function.prototype.sapply = function(context) {
  // 传进来的上下文
  let self = context || window;
  // 函数的返回值
  let result;
  // 给当前上下文添加属性 - 当前函数
  self.fn = this;
  // 执行函数 展开数组参数
  /**
   * arguments 包含了传进来的上下文以及参数
   * 当前例子
   * arguments [{title: 'nate', fn}, ['baoqi', 24]]
   * 这里要对 arguments做容错处理
   */
  if(arguments[1]) {
    result = self.fn(...arguments[1]);
  }else {
    result = self.fn();
  }
  // 删除添加的属性
  delete context.fn;
  // 返回函数执行的结果
  return result;
}

// 以上述person和foo为例
foo.sapply(person, ['baoqiwang', 18]) // baoqiwang 24 nate undefined
```

### 手动实现call

call的实现原理和apply基本是一致的，不过还是有点细微的差异

```javascript
Function.prototype.scall = function(context) {
  let self = context || window;
  let result;
  self.fn = this;
  /**
   * 差异 跟apply
   * typeof arguments  "object" arguments 是一个对象
   * args 是获取传入函数的参数数组
   */
  const args = [...arguments].slice(1);
  if(args.length) {
    result = self.fn(...args);
  }else {
    result = self.fn();
  }
  delete self.fn;
  return result;
}
```

接下来就开始扯一扯bind了

### bind

bind的内部运行原理基本和apply、call是类似的，但是他是返回了一个函数, 先看下原生的bind

```javascript
let newFoo = foo.bind(person)
newFoo('baoqi', 18) //  baoqiwang 24 nate
```

接下里手动实现bind, bind 可以接受多个参数

### 手动实现bind

1. 原型链的方式
2. 封装function的方式

```javascript
// 1. 原型链的方式
Function.prototype.sbind = function(context) {
  // let context = context || window;
  // 因为返回函数 闭包原因 这里保存函数引用 或者 返回函数采用箭头函数模式
  let self = this;
  // 柯里化传参 保存bind第一个后面的参数
  let args = [...arguments].slice(1);

  return function() {
    // 合并参数
    return self.apply(context, [...args, ...arguments])
  }
}
// 2. 封装function方式 这种方式就没有上述方式周密了
function sbind(fn, obj) {
  return () => {
    return fn.apply(obj, [...arguments])
  }
}
```

