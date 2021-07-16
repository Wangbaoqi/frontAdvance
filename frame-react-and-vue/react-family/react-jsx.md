# React  API

React 中导出了很多的API，这里对这些API结合例子来进行学习

下面是根据React源码（version 17.0.2）整理出来的常用的API。

![](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/react/React%20API%20.png)

接下来通过API的使用来更好的理解源码。

### React.Component

React.Component 是定义组件的基类，通过使用ES6语法来继承它来创建子组件。

有关 React.Component 的源码可以到[「React17 源码 - React API」](../../source-code-yuan-ma/react-17-yuan-ma/react-api.md#component-he-purecomponent)查看。

有关 React.Component 的详细使用可以到 [「React 官网」](https://zh-hans.reactjs.org/docs/react-component.html) 查看

```typescript
class ClassCom extends React.Component<Props, State> {
  constructor(props) {
    super(props)
  }
  render() {
    return <div>...</div>
  }
}
```

#### 组件的属性

类组件除了除了实例生命周期钩子之外，还有其他的API

* props
* state
* setState\(\)
* forceUpdate\(\)
* defaultProps
* displayName

#### **生命周期钩子调用**

[React组件生命周期 速查表](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

**页面挂载**

* **constructor\(\)**
* static getDerivedStateFromProps\(\)
* **render\(\)**
* **componentDidMount\(\)**

**页面更新**

* static getDerivedStateFromProps\(\)
* **shouldComponentUpdate\(\)**
* **render\(\)**
* **getSnapshotBeforeUpdate\(\)**
* **componentDidUpdate\(\)**

**页面卸载**

* **componentWillUnmount\(\)**

**生命周期钩子函数注释**

```typescript
class ClassCom extends React.Component<Props, State> {
  constructor(props) {
    super(props)
  }
  // 适用于罕见的用例 目的只有一个 当props变化时更新state
  // state 的值任何时候都取决于Props
  // init 和 update 都会调用
  // 派生State 以及 Test Case
  static getDerivedStateFromProps(props: Props, state: State) {
    // console.log(props, state, 'getDerived props');
    return null
  }

  // 默认行为是 state 或者 props 每次发生变化 都会重新渲染 大部分遵循默认行为
  // ! 注意 不要依靠此方法阻止渲染
  // return false 就会阻止渲染 从而不会调用 render componentDidUpdate
  // ! 后续版本可能会视为提示命令 从而return false 也会渲染
  shouldComponentUpdate(nextProps: Props, nextState: State) {
    // console.log(nextProps, nextState, 'shouldComponentUpdate props');
    if(nextState.name !== this.state.name) {
      return false
    }
    return true
  }
  render() {
    return <div>...</div>
  }
  // 在最近一次的渲染之前调用 在组件发生更改之前获取DOM的信息
  // 将返回值作为参数传给componentDidUpdate
  // 应用场景： UI处理中 以特殊方法处理滚动位置
  getSnapshotBeforeUpdate(preProps: Props, preState: State) {
    if(preState.list.length < this.state.list.length) {
      const box: any = this.listRef.current;
      return box?.scrollHeight - box?.scrollTop
    }
    return null
  }

  // 更新后立即调用 首次渲染不会调用
  componentDidUpdate(preProps: Props, preState: State, snapShot: any) {
    if(snapShot) {
      const box: any = this.listRef.current;
      box.scrollTop = box?.scrollHeight - snapShot
    }    
  }
}
```

### React.pureComponent





