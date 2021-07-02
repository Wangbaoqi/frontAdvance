# JS 基础 - 引用类型

在ECMAScript中，引用类型是一种数据结构，而**引用类型的值**是引用类型的一个**实例。**这样理解，引用类型跟**类**的概念差不多，是将**数据**和**功能**组织到一起的一种结构。但是严格意义上讲，引用类型并不能称之为类，虽然JavaScript是一门面向对象的语言，但是并没有提供类、接口的有关数据结构。

尽管ES6提出了关键字**Class**来更直观的看到面向对象的实现，但是根本意义上来讲，JavaScript是没有类的，只不过**Class**是语法糖而已。

```javascript
let obj = new Object();
```

上面就实现了一个简单的引用类型的实例，关键字 **new** 后面跟一个构造函数 **Object** 可以进行实例化。除了Object 引用类型，ECMAScript中可以将引用类型分为 基本引用类型 和 集合引用类型

### 基本引用类型

基本引用类型也是原生引用类型，是可以直接使用的，比如说常见的 [**Date** 日期](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)引用类型、[RegExp 正则](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)、原始类型包装引用类型（**Number**、**String**、**Boolean）** 以及单体内置对象（[**Global**](https://developer.mozilla.org/en-US/docs/Glossary/Global_object)、**Math**）

Global 对象是JavaScript中最特殊的一个对象，因为它是不存在的，只能访问到它的属性。

![Global &#x5C5E;&#x6027;](../../.gitbook/assets/global-object%20%281%29.png)

每个宿主环境（browser 和 Node）都存在Global对象，因此这些属性可以直接在全局中访问。

### 集合引用类型

集合引用类型有 [Object](object.md) 、[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)、[TypeArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)、Map、WeakMap、Set 和 WeakSet，后四种是ES6新增的引用类型。

有关Object、Array等经常使用的引用类型后面章节会专门涉及。这里阐述一下ES6新增的四种类型

### Map





