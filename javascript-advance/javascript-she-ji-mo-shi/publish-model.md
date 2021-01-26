# 发布-订阅模式

> 发布-订阅模式又叫做观察者模式，定义对象间的一对多的依赖关系，当一个对象的状态发生变化时，所有依赖他的对象都将得到通知

**发布-订阅作用**

* 可以应用异步编程中，替代传递回调函数的方案
* 取代对象对象之间硬编码的通知机制

### DOM事件

发布-订阅模式也在DOM事件上有一定的体现。我们不知道使用者何时点击dom元素，因此我们会将事件绑定到元素上。例如下面常见的例子

```javascript
document.body.addEventListener('click', function() {
  console.log('click it')
})
```

### 自定义事件

如何实现发布-订阅模式

* 指定发布者 
* 给发布者添加缓存列表，存放回调函数一遍通知订阅者
* 发布消息时，遍历缓存列表，触发订阅者的回调函数

实现一个简单的售楼处的例子

```javascript
// 定义发布者
var saleOffices = {};
// 给发布者添加订阅者缓存者列表
saleOffices.clientlists = [];
// 添加订阅者
saleOffices.listen = function(fn) {
  this.clientlists.push(fn)
}
// 发布者发布消息
saleOffices.trigger = function() {
  var fn = this.clientlists
  for(var i = 0; i < fn.length; i++) {
    fn.apply(this, arguments)
  }
}
// 测试 添加订阅者
saleOffices.listen(function(price) {
  console.log(+price)
})
saleOffices.listen(function(price) {
  console.log('first person sub msg'+price)
})
saleOffices.trigger(300)
saleOffices.trigger(230)
```

虽然实现了简易的模型，但是问题是该订阅会展示所有订阅的消息，并不能收到自己订阅的消息。加以改进

```javascript
// 添加订阅者
saleOffices.listen = function(key,fn) {
  if(!this.clientlists[key]) {
    this.clientlists[key] = []
  }
  this.clientlists.push(fn)
}
// 发布者发布消息
saleOffices.trigger = function() {
  var fn = this.clientlists
  var key = Array.prototype.shift.call(arguments)
  var fns = fn[key]
  for(var i = 0; i < fns.length; i++) {
    fns[i].apply(this, arguments)
  }
}
```

### 发布订阅的通用实现

可以将发布-订阅的功能提取出来，这样可以在多个楼盘中复用此订阅-发布功能

```javascript
var observer = {
  // 订阅缓存列表
  clientlists: {},
  // 订阅者加入缓存中
  listen(key, fn) {
    if(!this.clientlists[key]){
      this.clientlists[key] = []
    }
    this.clientlists[key].push(fn)
  },
  // 发布者发布消息
  trigger() {
    var key = Array.prototype.shift.call(arguments)
    var fns = this.clientlists[key]
    for(var i = 0; i < fns.length; i++){
      fns[i]()
    }
  }
}
// 可以给所有对象动态安装发布-订阅功能
function installPublish(obj) {
  for(var key in observer) {
    obj[key] = observer[key]
  }
}
// 定义发布者
var saleOffices = {}
// 给发布者 saleOffices 安装订阅-发布消息
installPublish(saleOffices)
```

### 取消订阅的事件

```javascript
var observer = {
  // ...
  remove(key, fn) {
    var fns = this.clientlists[key]
    // 没有此订阅者订阅的消息
    if(!fns) {
      return `没有${fns}订阅的消息`
    }
    // 此订阅者没有说明取消具体的订阅事件，则将所有订阅事件取消掉
    if(!fn) {
      fns && (fns.length = 0)
    }else {
      // 说明取消具体的订阅事件
      var subIndex = fns.findIndex(fn)
      fns.splice(subIndex, 1)
    }
  }
}
```

改进之后完整的[发布订阅](https://wangbaoqi.github.io/nateCase/designModel/publish/index.html)

### 小结

发布-订阅模式的使用非常广泛，可以用在异步编程中，也可以帮助完成松耦合的代码编写。 在架构方面来看，无论[MVC](/frame/mind/mvvm.html#mvc架构模式)、[MVP](/frame/mind/mvvm.html#mvp模式架构)和[MVVM](/frame/mind/mvvm.html#mvvm架构模式)中，都会有观察者模式的参与

发布-订阅模式的优点：

* 时间上的解耦
* 对象间的解耦

发布-订阅模式的缺点:

* 创建订阅者消耗一定的时间和内存
* 过渡使用，可能会弱化对象之间的关系，导致难以维护和理解

