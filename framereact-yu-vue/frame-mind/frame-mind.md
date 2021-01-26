# 框架思想

### 框架的好处

以前没有框架的时候，前端页面基本都是由服务端来处理的，而渲染是采用的各种的模板引擎。而这种模式随着现在技术的更新，业务的完善以及用户体验的提高，基本上快被弃用了。由此各种的框架由此产生，比如，现在使用最高的React、Vue以及angular，但是，使用他们能够解决什么问题呢?

* 组件化
* 强大的开源社区
* 强大的第三方插件
* 拥有各种工具帮助解决问题

看似这些特性都能算是使用框架的原因，但是，这只是使用框架的好处，其真正解决的问题是什么呢?

**解决了UI和状态同步的问题**

在没有框架的时代，要重新更新页面，必须要操作DOM，这就意味着要达到页面更新，必须要损失一定程度的性能损耗，除此之外，在代码维护以及扩展性就变的异常困难了。

在使用框架中，是怎么解决这个问题的呢 举例：

当状态改变后，映射出对应DOM的虚拟DOM，最终映射改变真实DOM，达到view层重新渲染

### 框架架构模式

架构模式是采用多种设计模式来改进代码结构的方式。实现软件设计的低耦合高内聚（降低模块间的复杂度，每个模块尽可能完成自己的功能）

框架模式产生的过程

1. 前后端不分的时代，前端代码由后端生成发送到浏览器（服务器渲染），最先采用的MVC模式，前端只是view层
2. 随着ajax的产生，前端有了获取数据的能力，不在依赖于服务器渲染
3. 随着前端的发展，用户体验的提升，前端也需要生成视图，处理大量的逻辑，导致前端的MVC、MVP以及MVVM模式的产生

### MVC架构模式

![mvc](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/frame/mvc.png)

如图，箭头代表事件调用，圆箭头代表事件通知

Model层:

* 用来封装数据以及处理数据的方式，数据变化通知view层
* model和view之间使用观察者模式，view注册监听model，更新view

View层:

* 数据的展示
* view和controller之间使用策略模式，view通过controller实例，实现特定的相应策略

Controller层:

* model和view之间的纽带，响应机制封装在controller层

简单实现MVC模式的demo，[测试地址](https://wangbaoqi.github.io/nateCase/mvcFront/mvc.html)

```javascript
// Model层 
// 数据封装、操作数据以及使用观察者模式
function Model() {
  var val = 0;
  this.add = function() {
    val++
  }
  this.getVal = function() {
    return val
  }

  // 观察者模式
  var views = []
  // 将观察者存入数组
  this.regist = function(view) {
    views.push(view)
  }
  // 通知view更新
  this.notify = function() {
    for(var i = 0; i < views.length; i++) {
      if(views[i].render) {
        views[i].render(this)
      }
    }
  }
}
// View 层 
// 页面显示 以及使用策略模式 
function View(controller) {
  var $show = document.getElementById('show')
  var $btn = document.getElementById('btn')

  this.render = function(model) {
    $show.innerText = model.getVal()
  }
  // view 和 controller之间使用策略模式，将Controller的方法绑定到元素的事件上
  $btn.onclick =  controller.add
}
// Controller层
// 初始化数据 
function Controller() {
  var model = null;
  var view = null;

  this.init = function() {
    model = new Model();
    view = new View(this);
    model.regist(view);
    view.render(model);
  }
  this.add = function() {
    model.add()
    model.notify()
  }
}

(function(){
  var controller = new Controller()
  controller.init()
})()
```

**使用MVC的弊端**

* 维护成本高。当页面复杂的时候，存在多个view，同时也要更新对应的model和Controller层
* 复用性差。同一套model层到代码，并不能满足多种页面渲染的方式
* 数据流混乱。改变model的同时，view层的更新触发了另一个Controller，使得另一个model又改变了

而随着这些问题的出现，MVP模式随之面世了。

### MVP模式架构

![mvc](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/frame/mvp.png)

如图, 跟MVC模式不同，model层跟view之间没有任何联系了，而所有的操作都放在了Presenter中

Model层:

* 仍然是封装数据和处理数据的方法
* view的更新不依赖于model，model负责管理数据
* model处理完数据将结果告知presenter

View层:

* 仍然是页面展示
* view和Presenter之间采用观察者模式以及策略模式

Presenter层:

* 作为model和view的中间人，除了逻辑操作外，还有同步数据

简单实现MVP模式的demo，[测试地址](https://wangbaoqi.github.io/nateCase/mvpFront/index.html)

```javascript
function Model() {
  var val = 0;
  this.add = function() {
    val++
  }
  this.getVal = function() {
    return val
  }
}

function View() {
  var $show = document.getElementById('show')
  var $btn = document.getElementById('btn')

  // 渲染页面
  this.render = function(model) {
    $show.innerText = model.getVal()
  }

  this.init = function() {
    var present = new Presenter(this)

    // 策略模式
    $btn.onclick =  present.add
  }
}

function Presenter(view) {
  var _model = new Model();
  var _view = view;

  // 初始化渲染
  _view.render(_model)


  this.add = function() {
    _model.add()
    _view.render(_model)
  }
}

(function(){
  var view = new View()
  view.init()
})()
```

**使用MVP的弊端**

MVP模式虽然将view和model解耦了，解决了MVC模式弊端中的2、3点，但是presenter中还是要同时兼顾view和model的问题并没有得到解决:

* Presenter中有大量的手动操作代码，使得Presenter变得很臃肿
* Presenter要处理多个视图，不好维护

### MVVM架构模式

![mvvm](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/frame/mvvm_model.png)

如图，MVVM将数据同步化，MVP中的Presenter是手动进行数据同步，MVVM中则是采用数据绑定进行同步数据

Model层:

* 只关注数据本身，没有任何数据动作，数据处理交给VM 

View层:

* 通过模板语法将数据渲染进DOM，通过数据绑定与Model进行数据同步 

ViewModel层:

* 除了实现业务逻辑之外，主要就是数据绑定，Presenter中的手动操作变成了VM中的数据绑定绑定来做

双向数据绑定:

* 实现view和model的同步自动化 Vue（数据劫持+观察者模式），angular（脏检查）

实现简易的MVVM架构模式

```javascript
// observer 监听每个数据 
function observe(data) {
  if(!data || data === 'undefined' || typeof data !== 'object') return 

  Object.keys(data).forEach(key => {
    defineReactive(data, key, data[key])
  })
}
// 数据劫持 
function defineReactive(data, key, val) {
  observe(val)
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    set(newVal) {
      val = newVal
      console.log('changed value');
    },
    get() {
      return val
    }
  })
}
```

**模板渲染脚本**

Vue的使用HTML模板

```markup
<div id="app">
  <input type="text" v-model="someStr">
  <input type="text" v-model="child.someStr">
  <button v-on:click="clickBtn">change model</button>
</div>
```

Vue的实例创建

```javascript
// 入口脚本
var vm = new MVVM({
  el: '#app',
  data: {}
})
// 创建MVVM
function MVVM(options){
  this.$options = options
  var data = this._data = options
  // 模板渲染处理
  new Compile(options.el || document.body, this)
}
```

在使用Vue时，会有挂载元素的标识`el`，获取该节点下的所有节点元素，对所有元素进行遍历处理，接下来看模板编译的简单示例

```javascript
// compile
function Compile(el, vm){
  this.$vm = vm; 
  // 获取挂载元素节点
  this.$el = this.isElementNode(el) ? el : document.querySelector(el);

  if(this.$el) {
    // 将原生节点拷贝到虚拟节点对象上
    this.$fragment = this.nodeFragment(this.$el)
    // 初始化虚拟元素
    this.initFragment()
  }
}

Compile.prototype = {
  constructor: Compile,
  // 原生节点拷贝虚拟节点
  nodeFragment(el) {
    // 创建虚拟对象节点
    var fragment = document.createDocumentFragment()
    var childNode = null
    // 拷贝原生节点到虚拟节点对象
    while(childNode = el.firstChild) {
      fragment.appendChild(childNode)
    }
    return fragment
  },
  // 初始化虚拟节点
  initFragment() {
    // 编译虚拟元素 遍历虚拟元素 处理绑定的指令和事件
    this.compileFragment(this.$fragment)
  },
  // 
  compileFragment(el) {
    var that = this;
    // 获取节点下的所有子节点集合
    var childNodes = el.childNodes;
    // 遍历子节点结合
    [].slice.call(childNodes).forEach(function(node) {
      // 获取子节点文本内容
      var text = node.textContent;
      // 正则匹配 {{}} 语法
      var reg = /\{\{(.*)\}\}/;
      // 元素节点
      if(that.isElementNode(node)) {
        // 编译元素节点
        that.compileElement(node)
      }else if(that.isTextNode(node) && reg.test(text)) {
        // 编译文本节点 
        that.complieText(node, RegExp.$1.trim())
      }
      // 如果该节点还有子元素节点集合 递归遍历处理
      if(node.childNodes && node.childNodes.length){
        that.compileFragment(node)
      }


    })

  },
  // 编译元素节点
  compileElement(node) {
    var that = this;
    // 获取元素属性集合
    var nodeAttrs = node.attributes;
    // 遍历属性集合
    [].slice.call(nodeAttrs).forEach(function(attr) {
      // 获取属性name
      var attrName = attr.name
      // 判断是否是 v- 指令
      if(that.isDirective(attrName)) {
        // 获取指令的 value值 例如 someStr
        var attrValue = attr.value;
        // 获取指令的 末尾值 例如 model
        var dir = attrName.substring(2)
        // 事件指令
        if(that.isEventDirective(dir)) {
          // 处理事件指令
          compileUtil.eventHandle(node, that.$vm, attrValue, dir)
        }else {
          // 普通指令 v-
          compileUtil[dir] && compileUtil[dir](node, that.$vm, attrValue)
        }

        node.removeAttribute(attrName)
      }

    })

  }
  // 是否是元素节点
  isElementNode(node) {
    return node.nodeType == 1
  },
  // 是否是v- 指令
  isDirective(attr) {
    return attr.indexOf('v-') == 0
  },
  // 是否是事件指令 on- 指令
  isEventDirective(dir) {
    return dir.indexOf('on-') == 0
  }

}
// 指令处理集合 
var compileUtil = {

  bind: function(node, vm, exp, dir) {
    var updateFn = updater[dir + 'Updater']
    updateFn && updateFn(node, this._getVmValue(vm, exp))

    new Watcher(vm, exp, function(value, oldValue){
      updateFn && updateFn(node, value, oldValue)
    })
  },
  // 数据绑定处理
  model: function(node, vm, exp) {
    // 数据绑定
    this.bind(node, vm, exp, 'model')

    var that = this;
    var val = this._getVmValue(vm, exp)

    node.addEventListener('input', function(e) {
      var newVal = e.target.value
      if(newVal = val) return
      that._setVmValue(vm, exp, newValue)
      val = newVal
    })

  },
  // 事件绑定处理
  eventHandle: function(node, vm, exp, dir) {
    // 获取事件类型
    var eventType = dir.split(':')[1]
    // 获取事件方法
    var eventFn = vm.$options.methods && vm.$options.methods[exp]
    // 该节点订阅事件方法
    if(eventType && eventFn) {
      node.addEventListener(eventType, eventFn.bind(this), false)
    }
  }
  // 获取VM的值
  _getVmValue(vm, exp) {
    var valVm = vm;
    exp = exp.split('.')
    exp.forEach(function(item) {
      valVm = valVm[item]
    })
    return valVm
  },
  // 设置VM的值
  _setVmValue(vm, exp, newVal) {
    var valVm = vm;
    exp = exp.split('.')
    exp.forEach(function(k, i) {
      if (i < exp.length - 1) {
          val = val[k];
      } else {
          val[k] = value;
      }
    })
  }
}

var updater = {
  modelUpdater: function(node, value, oldValue) {
    node.value = typeof value === 'undefined' ? '' : value
  }
}
```

原生元素和虚拟节点对象的对比, 拷贝到虚拟节点上之后

```markup
<!-- el 原始节点-->
<div id="app"></div>

<!-- fragment 虚拟节点-->
<input type="text" v-model="someStr">
<input type="text" v-model="child.someStr">
<button v-on:click="clickBtn">change model</button>
```

完整的简易MVVM模式-Vue [测试地址](https://wangbaoqi.github.io/nateCase/mvvmFront/index.html)

