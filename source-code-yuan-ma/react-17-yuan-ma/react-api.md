# React API

**ReactJs** 是纯粹的一个JavaScript库，可以使用它来将页面元素转换成对象，而页面渲染则是通过**ReactDOM**来操作的，这里归纳一下React API以及底层是如何运行的。

![](../../.gitbook/assets/react-api-.png)

在React中创建元素一般都是使用 **JSX** （JavaScript语法的扩展），即可以在**JavaScript**中使用 HTML 元素。而这样的语法格式，React是如何将它转换成对象的呢？答案肯定是**Babel** 转义。

首先看一段JSX代码

```javascript
const Foo = () => {
	return (
    <div>Component </div>
  )
}
<Foo />
```

上述定义了一个简单的函数式组件，看babel是将它转义成啥了。

```javascript
"use strict";

var Foo = function Foo() {
  return /*#__PURE__*/React.createElement("div", null, "Com ");
};

/*#__PURE__*/
React.createElement(Foo, null);
```

可以看到，当使用JSX时，React会调用 `createElement` 来创建元素，`createElement` 内部调用 `ReactElement` 来创建元素。

### ReactElement

这里先附上源码，这里源码的版本是 `17.0.3`  ，下面列举了主要的代码

```javascript
function createElement(type, config, children) {

  // 处理参数
  
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}


const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  return element;
};
```

从 **babel** 转义的结果看到，`createElement` 接收三个参数

* type
* config
* children





### Component 和 PureComponent

我们在日常开发定义的类组件都会继承 Component 或者 PureComponent，下面来看下这两个顶级的类内部是怎么实现的

```javascript
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // ?* 更新队列
  this.updater = updater || ReactNoopUpdateQueue;
  this.refs = {};
}

Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
}

Component.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate')
}
```

可以看到，Component 的定义很简单，原型上实现了常用的 `setState` 的方法，以及 `forceUpdate` 方法，而这两个方法更新 state 会进入 Fiber 调度阶段，这个后面再述。

```javascript
function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  // ?* 更新队列
  this.updater = updater || ReactNoopUpdateQueue;
  this.refs = {};
}

function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;
// TODO why this 
const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy())
pureComponentPrototype.constructor = PureComponent;

Object.assign(pureComponentPrototype, Component.prototype)
pureComponentPrototype.isPureReactComponent = true;
```









