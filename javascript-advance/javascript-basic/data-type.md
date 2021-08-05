# JS 基础 - 数据类型

> ECMAScript 语言中所有的值都有一个对应的语言类型。ECMAScript 语言类型包 括 Undefined、Null、Boolean、String、Number 和 Object。

### 内置类型

**JavaScript 中有七种内置类型**

* null
* undefined
* number
* string
* boolean
* object
* symbol \(ES6 新增\)
* bigInt 

#### typeof 判断类型

可以使用typeof运算符来判断值的类型, 返回字符串类型

```javascript
typeof null; // "Object“
typeof undefined; // "undefined"
typeof 10; // "number"
typeof "10"; // "string"
typeof {}; // "object"
typeof function(){}; // "function"
typeof [1,3]; // "object"
typeof Symbol(); // ”symbol“
```

**因此使用typeof并不能准确的判断所有的类型**

**undefined和undeclared**

undefined 和 undeclared（未声明）不是同一个概念；undefined是已经声明了，并未赋值；而undeclared是没有声明，已经使用了 

```javascript
var a;
typeof a; // undefined

console.log(b); // ReferenceError： b is not defined
typeof b; // undefined
```

#### instanceof 判断类型

判断类型也可以使用instanceof，其原理是通过原型链来判断的

```javascript
let obj = new Object();
obj instanceof Object; // true

[] instanceof Array; // true

null instanceof Object; // false

'123' instanceof String; // false

123 instanceof Number; // false

let str = new String('str');
str instanceof String; // true
```

**因此instanceof是不能判断基本类型的值，除了 hack 的方式**

```javascript
class CheckIsNumber {
  static [Symbol.hasInstance](number) {
    return typeof number === 'number'
  }
}
1 instanceof CheckIsNumber
```

#### constructor 构造函数判断类型

使用构造函数来判断

null和undefined在判断的时候会报错, 这种方法知道就好，不建议使用

```javascript
let obj = {};
obj.constructor === Object
[].constructor === Array
'22'.constructor === String
22..constructor === Number
```

#### toString 判断类型

toString 方法可以算是判断类型的终极解决方法

```javascript
Object.prototype.toString.call(333) // "[Object Number]"
Object.prototype.toString.call("hello") // "[Object String]"
Object.prototype.toString.call(false) // "[Object Boolean]"
Object.prototype.toString.call({name: 'xiaofan'}) // "[Object Object]"
Object.prototype.toString.call(null) // "[Object Null]"
Object.prototype.toString.call(undefiend) // "[Object Undefiend]"
Object.prototype.toString.call(function(){}) // "[Object Function]"
Object.prototype.toString.call([1,2,4]) // "[Object Array]"
```

### 类型的值

#### 数组和字符串

> 数组的和字符串很相似，两者都有共同的属性和方法，length，concat\(\), indexOf\(\)

```javascript
// 字符串反转
var a = '12345';
var b = a.split('').reverse().join('')
```

### 类型转换

JS类型转换的情况大致有三种： 

     1. 转换成boolean 

     2. 转换成string 

     3. 转换成number

**类型转换表格**

| 原始值 | 转换目标 | result |
| :--- | :---: | :--- |
| number | boolean | 0, -0, NaN为false， 其他为true |
| string | boolean | '' =&gt; false |
| undefined, null | boolean | false |
| 引用类型 | boolean | true |
| number | string | 2 =&gt; '2' |
| boolean, function, symbol | string | true =&gt; 'true' |
| array | string | \[\] =&gt; '' \[1,3\] =&gt; "1,3" |
| object | string | {} =&gt; "\[object object\]" |
| string | number | '' =&gt; 0 '2' =&gt; 2 |
| array | number | \[\] =&gt; 0   \[1\] =&gt; 1  \[1,2\] =&gt; NaN |
| null | number | 0 |
| 除了数组的引用类型 | number | NaN |
| symbol | number | 报错 |

#### 转换成Boolean

除了 null, undefined, false, 0, -0, '', NaN为false，其他都为true

#### 引用类型转原始类型

引用类型转原始类型会调用内置的函数\[\[ToPrimitive\]\],会经历一下过程：

* 已经是原始类型的话，不需要转
* 转换成字符串的话，调用toString,不是字符串就会调用valueOf，不是的话继续调用toString
* 没有原始类型的值，就会抛错

#### 四则运算

**+法运算符**

* 运算符一侧为字符串，就会把另一方转化为字符串
* 运算符一方不是数字或者字符串，就会转化为字符串或者数字
* 运算符一方是字符串或数字，另一方是引用类型，则会转换为字符串

```javascript
0 + '2'; // '02'
true + 0; // 1
3 + []; // '30'
0 + [1]; // '01'
2 + [1,2]; // '21,2'

'3' + []; // '3'
'3' + [1]; // '31'

3 + {}; // '3[object object]'
'3' + {}; // '3[object object]'
```

**+法运算符注意**

```javascript
+ 'n'; // NaN 
+ '1': // 1
'a' + + 'b'; // 'aNaN'
```

**除了加法运算符的其他运算符**

那么对于除了加法的运算符来说，只要其中一方是数字，那么另一方就会被转为数字

