# 数组原生API实现

### Array.prototype.filter 

```javascript
// this 是执行callback fn 中使用的this的值
Array.prototype.sfilter = function(fn, thisArg) {
  let self = thisArg || this;
  let arr = [];

  for(var i = 0; i < self.length; i++) {
    // callback 执行的结果
    /** param 
     * this[i] 当前的值
     * i 当前的位置索引
     * this 当前的引用的数组
     */
    fn(self[i], i, self) && arr.push(self[i])
  }
  return arr;
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

```javascript
Array.prototype.sreduce = function(fn, initVal) {
  for(let i = 0; i < this.length; i++) {
    initVal = fn(initVal, this[i], i, this)
  }
  return initVal
}
```

