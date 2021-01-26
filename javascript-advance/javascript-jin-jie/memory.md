# JS 进阶 - 内存与性能

### 栈空间 堆空间

JS的数据类型有7种，null、undefined、number、string、boolean、object、symbol。 大致分为基本类型（原始类型）和引用类型。

接下来看下这两种类型在内存里是如何存储的。

**JavaScript的代码在内存中有三种存在空间的方式:**

1. 代码空间 - 可执行代码的存在空间
2. 栈空间 - 调用栈的存在空间
3. 堆空间 - 引用类型的存在空间

![&#x5185;&#x5B58;&#x6A21;&#x578B;](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/JavaScript/memory/mem_1.png)

这是一段简单的code：

```javascript
function foo() {
  var a = 1;
  var b = '3';
  var c = {
    name: 'nate',
    age: 20
  }
  var d = c; // 两者引用堆空间的是同一地址
}
foo();
```

利用执行上下文分析foo函数中变量的存储方式

![&#x8C03;&#x7528;&#x6808;&#x548C;&#x5806;&#x7A7A;&#x95F4;](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/JavaScript/ctx/ctx_1.png)

可以看到，基本类型（原始类型）的数据基本都存在栈空间里，而引用类型数据在栈中只是其堆空间的地址，数据其实在堆空间中。

#### 闭包中的内存模型

看一段代码片段:

```javascript
function foo() {
  var name = 'nate.wang';
  var age = 18;
  var hobby = 'book';
  var updateName = {
    setName: function(newName) {
      name = newName
    },
    getName: function() {
      console.log(hobby)
      return name
    }
  }
  return updateName
}
var bar = foo()
bar.setName('wangbaoqi')
bar.getName() //
```

这是一段存在闭包的代码，再次利用执行上下文来分析一下：

1. foo\(\) 执行，产生foo函数执行上下文，变量name、age、hobby、updateName存在于变量环境对象中，updateName初始值为堆地址，其余初始值为undefined，在编译过程中，遇到setName函数，JS引擎对内部函数扫描一遍，进行了词法分析，发现内部函数引用了外部函数foo的变量name和hobby，因此引擎判断这是一个闭包，于是在堆空间了创建了闭包对象\(Clourse\(foo\)\)
2. foo函数执行完成，foo函数执行上下文栈从调用栈的顶部弹出，此时按理说foo函数里所有变量都会销毁，但是由于产生了闭包，变量name和hobby存在引用，并没有被销毁掉。

**foo函数执行到return的时候，调用栈是如何执行的**

![&#x95ED;&#x5305;-&#x6267;&#x884C;&#x6808;](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/JavaScript/ctx/ctx_2.png)

**总而言之：产生闭包Clourse的核心两步**

1. 预扫描内部函数 - 在编一阶段，JS引擎扫描代码进行词法分析
2. 内部函数\(setName\)内部引用外部函数的变量\(name, hobby\)保存在了堆空间中 

### 垃圾回收

**为什么要进行垃圾回收?**

JS中有些数据在被使用之后，可能就不需要了，因此不能让这块数据一直存在于内存当中，需要将这些垃圾数据回收掉。

JS中原始数据类型和引用数据类型的存在方式是栈空间和堆空间

在JS中，垃圾数据存在的方式相对应的也有栈中的垃圾和堆中的垃圾，接下来学习这两种垃圾是如何被回收的。

#### 代际假说和分代收集

代际假说是一个在垃圾回收领域中重要的术语，以后的垃圾回收策略都在此基础上。 其有两个特点： 1. 对象的存活时间很短，一经分配，很快就变的不可访问 2. 不死的对象，会存活的比较久

JavaScript引擎V8中会把堆空间分成新生代和老生代空间。

新生代空间: 存放的对象存活时间比较短，且分配的内存空间也比较小（1-8M），**由副垃圾回收器回收垃圾**

老生代空间: 存放的对象存活时间比较长，且分配的内存空间也比较大，**由主垃圾回收器回收垃圾**

#### 栈中的垃圾回收

栈中存在的数据是原始类型的数据，所谓的栈就是调用栈，当一段执行结束之后，其执行上下文会从调用栈中弹出，**此时会有记录当前调用栈中执行上下文的位置的指针 - ESP**，它会随着当前的执行状态而移动。就是在这个移动的过程中，就是销毁上个执行上下文的过程

```javascript
function foo() {
  var age = 18
  var b = { name: 'nate.wang', hobby: 'book'}
}
foo()
```

利用这个例子通过调用栈来阐述一下栈中的垃圾回收 ![&#x6808;&#x4E2D;&#x7684;&#x5783;&#x573E;&#x56DE;&#x6536;](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/JavaScript/ctx/ctx_3.png)

可以看到，当foo函数执行完成，ESP指针就会指向全局执行上下文，此时foo函数中的数据就会被回收掉。此时会有个问题，foo函数的执行上下文被弹出了，但是其对象b的引用还在堆空间中，这个就是堆空间的垃圾回收了

#### 堆中的垃圾回收

上述说到，foo函数的执行上下文被弹出了，但是对象b的引用还存在在堆中，这里堆中的垃圾回收就要用到垃圾回收器了。

上面讲到，垃圾回收器有主、副垃圾回收器

**垃圾回收器的工作流程**

1. 标记空间中的活动对象和非活动对象；活动对象 - 正在使用中的对象； 非活动对象 - 可以回收的对象
2. 回收非活动对象占用的内存
3. 内存整理 - 频繁回收对象之后，内存就会存在不连续的空间（内存碎片），内存整理就是将这些不连续的空间整理到一起 

**副垃圾回收器** 副垃圾回收器主要是用来处理新生代区域的垃圾回收。小的对象一般会在这个区域，垃圾回收的频率还是比较高的

**新生代区**采用的是**Scavenge算法**

![&#x65B0;&#x751F;&#x4EE3;&#x533A;](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/JavaScript/ctx/ctx_4.png) Scavenge算法是将新生代区分成两个区域：对象区域和空闲区域。 当对象区域写满时，副垃圾回收器会进行一下垃圾回收，对对象区域中的垃圾进行标记，然后将未标记的对象一次性复制到空闲区域，对标记的对象区域中的垃圾进行清理，然后将空闲区域和对象区域的角色进行互换。这样还能让新生代中的两块区域无限循环下去。

因为新生区的空间不大，很容易被存活的对象写满，因此有了**对象晋升策略:**如果两次垃圾回收还存活的对象，就会晋升到老生代区。

**主垃圾回收器**

主垃圾回收器主要是用来处理老生代区域的垃圾。一般存在在的对象都比较大，因此采用**Scavenge算法**是不合理的。

**老生代区**采用的是 **标记** - **清除**的算法

![&#x8001;&#x751F;&#x4EE3;&#x533A;](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/JavaScript/ctx/ctx_5.png)

首先是标记阶段，当ESP从foo执行上文下移到全局执行上下文，这时遍历调用栈，是不会找到0100的地址的，因此这块地址会被标记。 其次是清除阶段，主垃圾回收器会将标记的内存块进行清除，但是这样就会产生了不连续的内存，产生了内存碎片。 由于存在内存碎片，因此产生了**标记**-**整理**算法，将内存碎片通过算法整理成连续的内存空间

#### 全停顿

JavaScript是一门单线程的语言，因此垃圾处理过程和业务逻辑处理是互斥的，当开始垃圾回收的时候，JS脚本会暂停，直到垃圾回收处理完成，JS脚本才会继续进行。 这样的机制对于新生代区的垃圾回收影响不大，新生区空间比较小。但是对于老生代区来讲，这样操作会浪费很多的时间成本。

因此，V8引用有采用了一种**增量-标记**的算法，在垃圾标记的过程和JS脚本交替进行，直到标记阶段结束。这样一来，JS脚本就不会有很明显的视觉效果的卡顿了。

