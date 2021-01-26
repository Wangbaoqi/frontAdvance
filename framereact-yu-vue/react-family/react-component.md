# react  - component

### 理解React组件

![react-component](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/react/compoent.png)

* react组件可以说是内部维护的状态和外部传入的props组成的
* react组件也可以说是一个纯函数\(返回的值完全由传入的值决定\)
* react组件也遵循的是单项数据流

#### 如何创建一个React组件

**创建思路以及原则**

**创建思路**

1. 创建静态UI
2. 思考组件的状态组成 
3. 思考组件的交互方式

**创建原则**

* 单一职责原则\(每个组件只做一件事，组件变的复杂，可以拆分\)
* 尽量无状态原则\(组件所需数据尽量通过props传入，提高复用性，这种组件也可以说是 Pure 或者 Dumb 组件\)

**受控组件** 这两中一般在表单元素中使用的多 受控组件一般是由使用者控制维护 ![react-component](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/react/component_1.png)

非受控组件一般是由DOM自身维护 **和非受控组件** ![react-component](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/react/component_2.png)

#### React 组件的生命周期

在掌握的react的生命周期之前可以看下生命周期图谱

![react-component-lifeCycle](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/react/component_3.png)

**组件挂载**

在组件挂载的阶段\(组件被创建并且插入DOM树中\), 生命周期钩子函数调用的顺序

* [**constructor\(\)**](https://zh-hans.reactjs.org/docs/react-component.html#constructor)
* [static getDerivedStateFromProps\(\)](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
* [**render\(\)**](https://zh-hans.reactjs.org/docs/react-component.html#render)
* [**componentDidMount**](https://zh-hans.reactjs.org/docs/react-component.html#componentdidmount)

::: warning 注意： componentWillMount 钩子即将过时，避免使用 :::

**组件更新**

当组件的props或者state发生变化时，组件会重新渲染，其调用生命周期钩子函数的顺序

* [static getDerivedStateFromProps\(\)](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
* [shouldComponentUpdate\(\)](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)
* [**render\(\)**](https://zh-hans.reactjs.org/docs/react-component.html#render)
* [getSnapshotBeforeUpdate\(\)](https://zh-hans.reactjs.org/docs/react-component.html#getsnapshotbeforeupdate)
* [**componentDidUpdate\(\)**](https://zh-hans.reactjs.org/docs/react-component.html#componentdidupdate)

::: warning 注意： componentWillUpdate componentWillReciveProps 钩子即将过时，避免使用 :::

**组件卸载**

当组件从DOM树中移除时，调用此钩子函数

* \*\*[componentWillUnmount\(\)](https://zh-hans.reactjs.org/docs/react-component.html#componentwillunmount)

**错误处理**

当渲染过程，生命周期，或子组件的构造函数中抛出错误时，会调用如下方法

* [static getDerivedStateFromError\(\)](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromerror)
* [componentDidCatch\(\)](https://zh-hans.reactjs.org/docs/react-component.html#componentdidcatch)

### 容器组件 VS 展示组件

基本原则： 容器组件负责数据获取。展示组件根据props显示信息

优势： 1. 如何工作和如何展示分离 2. 重用性高 3. 更高的可用性 4. 更易用测试

```javascript
// CommentList 父组件组件
export default class CommentList extends Component {
  constructor(props) {
    super(props);

    this.state = {
      comments: []
    }
  }

  componentDidMount() {
    setInterval(() => {
      this.setState({
        comments: [
          {name: 'react', content: 'react components'},
          {name: 'vue', content: 'vue components'}
        ]
      })
    }, 1000)
  }
  render() {
    return (
      <div>
        {
          this.state.comments.map((v, i) => (
            // <Comment key={i} data={v}></Comment>
            <Comment key={i} {...v}></Comment>
          ))
        }
      </div>
    )
  }
}
```

组件CommentList的state comments 每隔一秒更新一次，组件Comment中的状态也会跟着变化，如果数据一致，就没有必要再次渲染Comment了，也就可以使用生命周期函数了 shouldComponentUpdate

```javascript
// 子组件
class Comment extends Component {
  // 可以使用PureComponent 代替
  shouldComponentUpdate(nextProps) {
    if(nextProps.data.name === this.props.data.name) {
      return false
    }
    return true
  }
  render() {
    console.log('render')
    const { name, content} = this.props
    return (
      <div>
        {name} - {content}
      </div>
    )
  }
}
```

### PureComponent

定制了shouldComponentUpdate后的component --- 浅比较

```javascript
class comp extends React.PureComponent {
  // TODO
}
```

改造上个组件Comment

```javascript
// 使用PureComponent
class Commentp extends React.PureComponent {
  render() {
    const { name, content} = this.props
    return (
      <div>
        {name} - {content}
      </div>
    )
  }
}
```

**原则**

* 确保数据类型是值类型
* 如果是引用类型，确保地址不变，或者确保不是深层次的嵌套 

**解决深层嵌套的方法：** 解套 {...obj}

### React.memo

React v16.6.0 之后的版本 可以使用一个新功能 React.memo来完美React 组件。让函数式的组件，也有了PureComponent的功能

