# React  API

React 中导出了很多的API，这里对这些API结合例子来进行学习

下面是根据React源码（version 17.0.2）整理出来的常用的API。

![](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/react/React%20API%20.png)

接下来通过API的使用来更好的理解源码。

### React.Component

React.Component 是定义组件的基类，通过使用ES6语法来继承它来创建子组件。

有关 React.Component 的源码可以到[「React17 源码 - React API」](../../source-code-yuan-ma/react-17-yuan-ma/reactdom/react-api.md#component-he-purecomponent)查看。

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

React.pureComponent 是纯组件，用法上跟React.Component基本一致，但是两个还是有一点细微的差别，在 shouldComponentUpdate 上的区别以及源码解析，详情可以到「[react 源码 - react API」](../../source-code-yuan-ma/react-17-yuan-ma/reactdom/react-api.md#component-he-purecomponent)查看.

### React.memo

React.memo 是高阶组件，接收两个参数 `type（React元素）`，`compare` - 新老props对比函数。

关于React.memo的底层实现原理可以到[「react 源码 - React API」](../../source-code-yuan-ma/react-17-yuan-ma/reactdom/react-api.md#react-memo)查看。

```typescript
const MyComponnet = React.memo(
  type: ReactElementType, 
  compare?: (oldProp, newProps) => boolean
)
```

React.memo 重点关注的是组件的渲染是否可以记忆化，这个就要求`props`是不是变化的，如果每次传入的`props`是相同的，其渲染的结果也是相同的，那么这个组件就可以被记忆化，就不要之后的`diff`操作了，也是提升性能的一种手段。

默认情况下（没有`compare`参数），在更新props的时候，对比新老props是使用的浅层对比，跟**shouldComponentUpdate**默认是否需要渲染采用的对比方式是一致的。

如果想手动控制对比过程，就可以采用第二个参数 `compare` 返回值必须是`boolean` 类型的。

但是不能任意的使用React.memo，否则会出现bug。

只要符合**组件接收同一个props，每次输出一致**就可以使用React.memo

#### 记忆化被破坏

```typescript
const Logout = (name, onLagout) => {
  
  return (
    <div onClick={onLogout}>
      Logout {name}
    </div>
  )
}
const MemoizedLogout = React.memo(Logout)
```

上面是一个登出和用户信息展示的组件，接收 `name` 和 `onLagout` ，被封装成了记忆化组件。但是父组件每次渲染的时候，Logout 组件也会渲染，没有达到记忆化组件的期望。

```typescript
const App = ({ store, cookies }) => {
  return (
    <div className="main">
      <header>
        <MemoizedLogout
          username={store.username}
          onLogout={() => cookies.clear('session')}
        />
      </header>
      {store.content}
    </div>
  );
}
```

这里存在一个问题，就是回调函数 `onLagout` ，函数本质就是对象，在浅层对比的时候，新老props是不相同的，所以会导致每次渲染。

不过这个可以用Hooks中的 `useCallback` 来解决。关于Hooks，在之后会涉及到。

```typescript
 const onLogout = useCallback(
   () => cookies.clear('session'), 
   [cookies]
 );
```

### React.createRef

refs 提供了一种方式，可以访问DOM节点或者React元素。

#### refs的主要用途：

* 管理焦点，文本选择或者媒体播放
* 触发强制动画
* 集成第三方DOM库

#### refs的使用方式

![](../../.gitbook/assets/refs.png)

#### refs的执行时机

在组件挂载时，给`current`属性传入DOM元素或者react元素（若是回调refs，会调用ref的callback函数并传入DOM元素，当卸载时传入`null`），当卸载时，给`current`传入null ，详细的请看 「[React源码 -  React API」](../../source-code-yuan-ma/react-17-yuan-ma/reactdom/react-api.md#react-createref)

#### 创建Refs

创建refs有以下几种方式

* string ref - 已废弃
* createRef 
* callback Ref
* useRef - 见hooks 

```typescript
class TestRef extends React.Component {
  // 将ref分配给组件的实例，方便整个组件引用它
  ref = React.createRef();
  // callback ref
  cbRef = null;
  
  constructor(props) {
    super();
  }
  
  /** callback ref 
   * @param(el): react元素或者DOM节点
   * 将el 存储在组件实例上
   **/
  refCall = (el) => {
    this.cbRef = el
  }
  
  componentDidMount() {
    console.log(this.ref.current, 'createRef instance')
    console.log(this.cbRef, 'callback Ref instance')
  }
  
  render() {
    return (
      <div>
        <input type="text" ref={this.ref} />
        <input type="text" ref={this.refCall} />
      </div>
    )
  }
}
```

#### 使用Refs

Refs的使用大致有跨层级的和不跨层级的，上述创建Refs的过程就是基于不跨层级的，下面就来看下跨层级的场景。

**传递给子组件 - createRef**

```typescript
// 类子组件 函数组件同理 也是从props获取
class Child extends React.Component {
  render() {
    return (
      <input type="text" ref={this.props.refInfo}/>
    )
  }
  componentDidMount() {
    if(this.ref) this.ref.current.focus();
  }
}


class App extends React.Component {
  ref = React.createRef(); 
  render() {
    return (
      <Child refInfo={this.ref}/>
    )
  } 
}
```

#### 传递给子组件 - callback Ref

```typescript

const Child = (props) => {
  return (
    <input type="text" ref={(el) => props.refInfo(el)} onChange={props.onChange}/>
  )
}

class App extends React.Component {
  refInput = null;
  ref = (el) => this.refInput = el; 
  
  changeText = () => {
    console.log(this.refInput?.value)
  }
  render() {
    return (
      <Child refInfo={this.ref} onChange={this.changeText}/>
    )
  } 
}
```

还有一种跨层级传递refs的方式 - **forwardRef 转发** 👇 

### React.forwardRef

Ref 的转发也是一种将ref传递到子组件的技巧。这在可重用的组件库中是很有用的。关于源码请到「[React 源码 - React API 」](../../source-code-yuan-ma/react-17-yuan-ma/reactdom/react-api.md#react-forwardref)

```typescript
const FancyButton = () => {
  return (
    <div className='fancyButton' ref={ref}>
      {props.children}
    </div>
  )
})
// 通过forwardRef转发
const FancyButtonNew = React.forwardRef((props, ref) => <FancyButton ref={ref} {...props}/>)

class ForwardRefs extends React.Component {
  ref = React.createRef<HTMLDivElement>();
  componentDidMount() {
    console.log(this.ref);
  }
  render() {
    return (
      <div>
        <p>Ref forward</p>
        <FancyButtonNew  ref={this.ref} />
      </div>
    )
  }
}
```

### React.lazy

React.lazy 允许定义一个可以动态加载的组件，这样可以减少bundle的体积，可以进行延迟加载初次未使用到组件

React.lazy 接收一个函数，该函数动态调用`import()`，必须返回一个`Promise`，该`Promise`需要一个`resolve`一个**`default`** `export` 的组件。然后必须应用在**Suspense**组件中。

```typescript
import React from 'react'

const CustomComponent = React.lazy(() => import('./Header'))
const LazyCom = () => {
  return (
    <React.Suspense fallback={'loading'}>
      <CustomComponent />
    </React.Suspense>
  )
}
```

如果动态引入组件出现异常，可以使用**边界异常捕获**来处理。

动态组件一般在路由中使用，这样在用户体验方面得到一定的提升。源码请看「React 源码 - React API」

### React.Suspense

React.Suspense 可以指定一个加载指示器，以防组件树中的子组件没有具备渲染的能力。

目前，懒加载组件是加载指示器的唯一用例

```typescript
import React from 'react'

const CustomComponent = React.lazy(() => import('./Header'))
const LazyCom = () => {
  return (
    <React.Suspense fallback={'loading'}>
      <CustomComponent />
    </React.Suspense>
  )
}
```

React.Suspense 接收`fallback`属性，其值就是加载指示器的内容。



