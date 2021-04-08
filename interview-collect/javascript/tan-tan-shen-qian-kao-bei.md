# 谈谈深浅拷贝

**拷贝是日常开发常用的方式，也是面试高频的问题**

### 浅拷贝

> 浅拷贝: 对基本类型来讲，就是值得拷贝，对于引用类型来讲，就是引用的拷贝，但是只会拷贝一层

**浅拷贝的方式**

1. Object.assign 

[MDN - Object.assign](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign):是将所有可枚举属性的值从一个或者多个源对象复制到目标对象，将返回这个目标对象。

```javascript
var obj = {
  name: 'nate.wang',
  friends: {
    name: 'baoqiwang',
    age: 20
  }
}
var newObj = Object.assign({}, obj)
console.log(newObj)

newObj.name = 'baoqi.wang'
newObj.friends.name = 'baoqi'

console.log(obj)
```

可以看到obj和newObj的friends的值都变了，因此，Object.assign只能拷贝引用类型的一层

1. Array.prototype.slice 

[MDN - Array.prototype.slice](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice):返回一个新的数组对象。新的数组对象由（begin和end - 不包含）决定的原数组的浅拷贝。原数组的值不会改变。

```javascript
var arr = [
  1,
  '2',
  {
    name: 'nate'
  }
]
var newArr = arr.slice(1) // ['2', {name: 'nate'}]
arr[2].name = 'baoqi'
console.log(arr) // [1, '2', {name: 'nate'}]
console.log(newArr) // ['2', {name: 'baoqi'}]
```

1. 扩展运算符 Spread

ES6 新增展开运算符，也可以实现浅拷贝

```javascript
var newObj = {...obj}
var newArr = [...arr]
```

1. 手动实现浅拷贝

实现浅拷贝的原理是遍历一层就可以

```javascript
function copy(obj) {
  if(typeof obj !== 'object' || obj === null) return 

  let newObj = Array.isArray(obj) ? [] : {}

  for(let i in obj) {
    if(obj.hasOwnProperty(i)) {
      newObj[i] = obj[i]
    }
  }
  return newObj
}
let newObj = copy(obj)
```

### 深拷贝

> 深拷贝： 顾名思义，重新拷贝一份，且两者互不联系

**深拷贝的方式**

1. JSON.parse\(JSON.stringify\(obj\)\)

```javascript
// Object 和 array 都可以完成深拷贝
var obj = {
  name: 'nate.wang',
  friends: {
    name: 'baoqiwang',
    age: 20
  }
}
var newObj = JSON.parse(JSON.stringify(obj))
obj.friends.name = 'baoqi.nate'

conosle.log(newObj)
// {
//   name: 'nate.wang',
//   friends: {
//     name: 'baoqiwang',
//     age: 20
//   }
// }
```

**注意：以下的方法会有问题**

* 忽略undefined 
* 忽略Symbol
* 忽略function - 不能序列化函数
* 不能解决循环引用的对象
* 不能处理 Date\(\)
* 不能处理正则

```javascript
var obj = {
  a: undefined,
  b: Symbol('su'),
  c: function(){},
  d: 30
}
var newObjF = JSON.parse(JSON.stringify(obj)) // {d: 30}

// 循环引用的问题
var objLoop = {
  a: 1, 
  b: {
    c: 2,
    d: 3
  }
}
objLoop.a = objLoop.b
objLoop.b.d = objLoop.a
var newObjL = JSON.parse(JSON.stringify(objLoop)) // Uncaught TypeError: Converting circular structure to JSON

// new Date() 的问题 不能正确转化
var date = new Date() // Tue Dec 24 2019 10:34:00 GMT+0800 (中国标准时间)
var newDate = JSON.parse(JSON.stringify(date)) // 2019-12-24T02:33:19.936Z

// 解决方法 转换成时间戳再拷贝
JSON.parse(JSON.stringify(+new Date()))

// 正则问题

var objReg = {
  a: 3,
  b: /'123'/
}
JSON.parse(JSON.stringify(objReg))
```

1. 递归实现深拷贝 

首先**实现浅拷贝**

```javascript
function cloneShadow(obj) {
  if(typeof obj !== 'object' || obj === null) return obj

  var target = Array.isArray(obj) ? [] : {}

  for(let key in obj) {
    if(Object.protoproto.hasOwnPerperty.call(obj, key)) {
      target[key] = obj[key]
    }
  }
  return target
}
```

稍微改动一下，可以进行深拷贝

[**MDN - WeakMap**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) 可以解决循环引用的问题

```javascript
function isObject(source) {
  return Object.prototype.toString.call(source) === '[object Object]'
}

function deepClone(source, weak = new WeakMap()) {
  if(!isObject(source)) return source
  if(weak.has(source)) return weak.get(source)

  let target = Array.isArray(source) ? [] : {}
  weak.set(source, target)

  for(let key in source) {
    if(Object.prototype.hasOwnProperty.call(source, key)) {
      target[key] = isObject(source[key]) ? deepClone(source[key], weak) : source[key]
    }
  }
  return target
}

var objs = {
  a: 1,
  b: '2',
  c: false,
  d: {
    name: 'baoqi'
  },
  e: [3,4, [5,6]],
  f: undefined,
  g: function() { console.log('function') },
  h: Symbol('symbol'),
  i: /'123'/
}

var newObjs = deepClone(objs)
```

### 破解递归爆栈

使用递归可能出现的情况就爆栈（如果深度比较深的话）。解决方法常见的有**消除尾递归**和**采用循环**

1. 消除尾递归
2. 采用循环的方式 不会出现爆栈的情景 

在这里使用循环的方式来破解递归爆栈

先附上code

```javascript
function deepLoop(source) {
  const root = {}
  // 栈顶的元素
  const loopList = [
    {
      parent: root,
      key: undefined,
      data: source
    }
  ]

  while(loopList.length) {
    const node = loopList.pop();
    const parent = node.parent;
    const key = node.key;
    const data = node.data;

    // 初始化
    let res = parent;

    if(typeof key !== 'undefined') {
      res = parent[key] = {}
    }

    for(let i in data) {
      if(data.hasOwnProperty(i)) {
        if(typeof data[i] === 'object') {
          loopList.push({
            parent: res,
            key: i,
            data: data[i]
          })
        }else {
          res[i] = data[i]
        }
      }
    }
  }
  return root;
}
```

这里采用了数据结构**栈**的方式。

结束循环的条件是栈为空。

### 参考

  \*  [深拷贝的终极探索](https://yanhaijing.com/javascript/2018/10/10/clone-deep/)

