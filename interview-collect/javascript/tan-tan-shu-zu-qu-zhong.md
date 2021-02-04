# 谈谈数组去重

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

