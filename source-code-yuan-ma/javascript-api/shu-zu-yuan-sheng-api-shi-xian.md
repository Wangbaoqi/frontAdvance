# 数组实现篇

## 原型API实现

数组[原型API](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)

### Array.prototype.indexOf

根据`indexOf` 的定义，这里实现了其`Polyfill`版本

```javascript
arr.indexOf(searchElement[, fromIndex])
```

接收两个参数  

* `searchElement` 查询目标值 
* `fromIndex` 从什么位置查
  * `fromIndex >= len`   不会再数组中查
  * `fromIndex` 为负值，如`-1`代表从数组末尾查找，依次类推。不过不会影响其查找顺序。
  * `len - Math.abs(fromIndex) < 0`   则还是查找整个数组，索引从`0`开始

返回值

* 查询有结果 返回其值索引位置
* 无结果 返回 -1

```javascript
Array.prototype.nIndexOf = function(target, fromIdx) {
  let idx;
  
  const arr = Object(this);

  const len = arr.length >>> 0;

  if(len == 0) return -1;

  // 将fromIdx 转换成 number 类型
  idx = +fromIdx || 0;

  if(Math.abs(idx) === Infinity) {
    idx = 0;
  }

  if(idx >= len) return -1;

  // 抵消idx 
  idx = Math.max(n >= 0 ? 0 : len - Math.abs(n), 0)

  while(idx < len) {
    if(idx in arr && arr[idx] === target) {
      return idx
    }
    idx++;
  }

  return -1
}
```

### Array.prototype.forEach

根据forEach的定义，实现其`Polyfill` 版本

```text
arr.forEach(callback(currentValue [, index [, array]])[, thisArg])
```

接收两个参数

* `callback` 数组中的每个元素需要执行的函数
  * `currentValue` 当前执行的元素值
  * `index 可选` 当前执行的元素的索引
  * `array 可选` 当前数组副本
* `thisArg 可选` 当执行回调函数时，用作`this`的值

返回值为`undefined` 

#### 注意事项

`forEach()` 按升序为数组中的每一项元素执行回调函数。

{% hint style="warning" %}
`thisArg` 如果有值，则回调函数中this指向其`thisArg`，如果未指定值，回调函数中`this`指向全局对象。

`forEach` 在遍历的第一次已经确定了其范围，也就是整个数组的值，在遍历的过程中，向数组添加的值（删除的值）不会被`callback`接收到。

**如果想要终止、跳出`forEach`循环，只能抛出异常**。
{% endhint %}

```javascript
Array.prototype.nForEach = function(cb, thisArg) {

  if(this == null) {
    throw new TypeError('this is null or defined')
  }

  let that = null;
  let idx = 0;
  // 
  let array = Object(this);
  let len = array.length >>> 0;

  if(typeof cb !== 'function') {
    throw new TypeError('callback is not function')
  }

  if(arguments.length > 1) {
    that = thisArg
  }

  while(idx < len) {
    let val

    if(idx in array) {
      val = array[idx]
      cb.call(that, val, idx, array)
    }
    idx++
  }
}
```

### Array.prototype.some  

根据**some**的定义，实现其`Polyfill` 版本

```text
arr.some(callback(element[, index[, array]])[, thisArg])
```

接收的参数跟forEach 一致，返回值为`true or false` 

```javascript
Array.prototype.nSome = function(cb, thisArg) {
  
  if(this == null) {
    throw new TypeError('this is null or defined')
  }
  
  if(typeof cb != 'function') {
    throw new TypeError('callback is not function')
  }
  let idx = 0, that = thisArg || void 0;
  
  const array = Object(this);
  const len = array.length >>> 0;
  
  while(idx < len) {
    if(idx in array && cb.call(that, array[idx], idx, array)) {
      return true
    }
  }
  return false;
}
```

### Array.prototype.filter 

```javascript
// this 是执行callback fn 中使用的this的值
Array.prototype.sfilter = function(cb, thisArg) {
  if(this == null) {
    throw new TypeError('this is null or defined')
  }
  
  if(typeof cb != 'function') {
    throw new TypeError('callback is not function')
  }
  let idx = 0;
  let that = thisArg || null;

  let array = Object(this)
  let len = array.length >>> 0;

  let result = []

  while(idx < len) {
    if(idx in array && cb.call(that, array[idx], idx, array)) {
      result.push(array[idx])
    }
    idx++
  }
  return result
}

// test 1
let filterArr = arr.sfilter((item, index, arr) => {
  return item > 4
}, arr)

// test 2
let ffArr = Array.prototype.sfilter.call(arr, (item, index, arr) => {
  return item > 4
})
```

### Array.prototype.map

```javascript
/**
 * fn callback
 * thisArg 可选参数
 */ 
Array.prototype.smap = function(fn, thisArg) {
  let self = thisArg || this;
  let arr = [];
  for(let i = 0; i < self.length; i++) {
    // callback 执行的结果被添加到新数组中
    arr.push(fn(self[i], i, self))
  }
  return arr;
}

let newArr = arr.smap((item, index, arr) => {
  return `${item}nate`
})
```

### Array.prototype.reduce

根据**reduce**的定义，实现其`Polyfill` 版本

```text
arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])
```

接收两个参数

* `callback` 数组中的每个元素需要执行的函数
  * `accumulator`  累加器累计回调的值，是上一次调用回调返回的累加值
  * `currentValue` 当前执行的元素值
  * `index 可选` 当前执行的元素的索引
  * `array 可选` 当前数组副本
* `initialValue 可选` 作为第一次调用 `accumulator` 的初始值

返回值为累加器结果 

```javascript
Array.prototype.nReduce = function(fn, initVal) {
  
  if(this == null) {
     throw new TypeError('this is null or defined')
  }
  if(typeof cb !== 'function') {
    throw new TypeError('callback is not function')
  }

  let array = Object(this)
  let len = array.length >>> 0;

  let idx = 0;
  let value;

  if(arguments.length >= 2) {
    value = initValue
  }else {

    while(idx < len && !(idx in array)) {
      idx++
    }

    if(idx >= len) {
      throw TypeError('Reduce of empty array with no initValue')
    }

    value = array[idx++]
  }

  while(idx < len) {
    if(idx in array) {
      value = cb(value, array[idx], idx, array)
    }
    idx++
  }

  return value
}
```

## 数组去重



数组去重已经是一个老生常谈的话题了，这里再重新温习和稳固一下，之前在[数据结构-数组常见使用场景](/algorithm/structure/array.html#数组去重)中简单学习过，这次全面的攻克掉。

这里收集了不同的数组去重的方法，接下来逐一去攻克。。

### ES6 Set数据结构去重

想必ES6去重应该在开发中使用的是最多的（简便明了）👍

这里使用了[MDN - Array.from\(\)方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from), 以及利用[Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)结构不能重复的特性来实现去重

```javascript
const arr = [1, 1, 2, 3, 5, 2, 9, 4, 8, 3, '1', '8', {}, {}];
// 方式 Array.from
Array.from(new Set(arr)) // [1, 2, 3, 5, 9, 4, 8, "1", "8", {}, {}]
// 方式 ... 扩展运算符
[...new Set(arr)] // [1, 2, 3, 5, 9, 4, 8, "1", "8", {}, {}]
```

可以看到，这种方式可以去重相同数据类型的值（基于基本类型），{} 对象也没有去重掉。况且兼容性也不太好。

### 双重循环去重

**利用splice ES5**

```javascript
for(var i = 0; i < arr.length; i++) {
  for(var j = i + 1; j < arr.length; j++) {
    // 如果有重复元素 删掉重复元素 标识从上一个开始
    // === 是否强制类型转换
    if(arr[i] === arr[j]) {
      arr.splice(j, 1);
      j--;
    }
  }
}
arr; // [1, 2, 3, 5, 9, 4, 8, "1", "8", {}, {}]
```

### indexOf去重

判断新数组中是否有该元素，没有就添加到新数组

```javascript
var newArr = [];
for(var i = 0; i < arr.length; i++) {
  if(newArr.indexOf(arr[i]) == -1) {
    newArr.push(arr[i])
  }
}

newArr; // [1, 2, 3, 5, 9, 4, 8, "1", "8", {}, {}]
```

### filter去重

利用[MDN - Array.prototype.filter](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)中的索引，第一次出现的位置跟第二次出现的位置是否一致。

这里提下[filter的原生实现](/algorithm/structure/array.html#array-prototype-filter)，可能更有助于理解这种方式

```javascript
function unique(arr) {
  return Array.prototype.filter.call(arr, (item, index) => {
    return arr.indexOf(item) === index
  })
}
unique(); // [1, 2, 3, 5, 9, 4, 8, "1", "8", {}, {}]
```

### Es6 Map数据结构去重

利用[MDN - Map数据结构](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)的键值是唯一的特性进行去重。

```javascript
function uniqueMap(arr) {
  let map = new Map()
  let resArr = []

  for(let i = 0; i < arr.length; i++) {
    if(map.has(arr[i])) {
      map.set(arr[i], true)
    }else {
      map.set(arr[i], false)
      resArr.push(arr[i])
    }
  }
  return resArr
}
uniqueMap(arr) // [1, 2, 3, 5, 9, 4, 8, "1", "8", {}, {}]
```

### 引用类型去重

上述的几种方法可以看到，对于基本类型可以去重，但是引用类型是无效的，在开发场景中，对于引用类型的去重也是非常常见的

```javascript
function uniqueArray(arr) {
  return [...new Set(arr.map(e => JSON.stringify(e)))].map(e => JSON.parse(e))
}
```

## 数组扁平化



## 数组之间交集





