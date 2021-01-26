# react - redux 手动实现

**这一节会在上一节的基础上加上专门适用于React的状态管理 -- redux + context = react-redux**

### Context 是什么

**context其实像就是组件树上某颗子树的全局变量**

### Context 的使用

```javascript
// 父组件 APP
import PropTypes from 'prop-types';

export class App extends Component {

  // 验证context的类型
  static childContextTypes = {
    theme: PropTypes.string
  }

  constructor() {
    super()
    this.state = {
      color: 'red'
    }
  }

  // 父组件中获取
  getChildContext() {
    return {
      theme: this.state.color
    }
  }

  render() {
    return (
      <div className='app'>
        <Header />
      </div>
    )
  }
}

// 子组件Header
class Header extends Component {

  // 子组件中获取 context的值 必须声明 contextTypes
  static contextTypes = {
    theme: PropTypes.string
  }

  render() {
    return (
      <div className='header'>
        <p style={{color: this.context.theme}}>this is a context theme.</p>
      </div>
    )
  }
}
```

### React-redux 的手写实现

先看一个使用react-redux实现的效果图, 业务需求是点击不同颜色的按钮，改变整个APP组件的主题颜色

![react-component](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/react/redux_1.png)

看下父组件的结构:

```javascript
import React, { Component } from 'react';
import {store} from './store'
import Provider from './provider'

// 将Header分成smart component 和 dumb component
import Header from './containers/Header'
import Content from './content'

export class ThemeIndex extends Component {
  render() {
    return (
      <div>
        <Provider store={store}>
          <Header headerProps={'headerhhh'}/>
          <Content/>
        </Provider>
      </div>
    );
  }
}
export default ThemeIndex;
```

父组件中引入store以及Provider\(集中处理全局变量context，将context传入子组件中\)

### React-redux 中store的实现

react-redux 中store的实现跟redux中store的实现是类似的，一下是code：

```javascript
// createStore 实现
export function createStore(reducer) {
  let listeners = [];
  let state = null;

  // 获取state
  const getState = () => state;
  // 订阅监听修改state的function
  const subscribe = (listener) => listeners.push(listener);
  // 修改state
  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  }
  // init state
  dispatch({});

  return { getState, subscribe, dispatch }
}
```

**createStore实现详解**

* 接受reducer\(function\)为入参，reducer-state初始化以及根据action的type修改state
* getState\(\) 获取state的值，也就是原始state
* subscribe\(\) 订阅监听修改state状态
* dispatch\(\) 专门修改state，listener监听state状态变化

### React-redux 中reducer的实现

前面实现了createStore的实现，因其调用了reducer,接下来实现reducer,以下是code:

```javascript
// action type
const THEME_COLOR = 'THEME_COLOR'

// reducer
const themeReducer = (state, action) => {
  if(!state) {
    return {
      themeColor: 'red'
    }
  }
  switch(action.type) {
    case: THEME_COLOR,
      return {
        ...state,
        themeColor: action.themeColor
      }
    default:
      return false
  }
}

// action creators
const changeThemeColor = (color) => {
  return { type: THEME_COLOR, themeColor: color }
}

export { themeReducer, changeThemeColor }
```

**reducer的实现详解**

* action type定义类型枚举
* themeReducer 返回一个新对象，里面需要修改的属性指针变了，未修改的属性指针还是指向原来的地址
* action creator 返回需要修改的type以及需要修改key值得value，也是diapatch的入参

### React-redux 中connect的实现

**引入connect的初衷**

在没有引入connect之前，子组件要拿到父组件传入的themeColor，每个子组件必须从context获取，这样就出现了两个比较严重的问题：

1. 很多相同的冗余的代码，从context中获取状态来设置自己的状态
2. 对于context的依赖太强，组件的复用性基本为零

**组件复用性最强：组件的状态来自外部的props和内部维护的状态，类似于纯函数一样，这样的组件复用性最强**

因此需要一种方式来从context获取状态，用这种方式统一将状态传入到pureComponent或者dumbComponent，提高组件的复用性，而这种方式可以称之为高阶组件。

**高阶组件：** 类似于纯函数一样，传入一个组件，进行复杂逻辑的处理，输出一个组件，传入的这个组件，其状态就会来自于外部传入的内部维护的，复用性极强。

而在这里，connect就充当着这种角色，它是把context和pureComponent连接起来了。

看下它的实现：

```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types'

export const connect = (mapStateToProps, mapDispatchToProps) => (WrappedComponent) => {
  class Connect extends Component {
    // 获取context
    static contextTypes = {
      store: PropTypes.object
    }
    constructor() {
      super();
      // 整合所有的状态
      this.state = {
        allProps: {}
      }
    }
    componentWillMount() {
      const { store } = this.context;
      // 更新所有的状态
      this._updateProps();
      // 订阅
      store.subscribe(this._updateProps())
    }
    // 更新allProps
    _updateProps() {
      const { store } = this.context;
      const stateProps = mapStateToProps  
                            ? mapStateToProps(state.getState())
                            : {};
      const dispatchProps = mapDispatchToProps 
                              ? mapDispatchToProps(state.dispatch()) 
                              : {};
      this.setState({
        allProps: {
          ...stateProps,
          ...dispatchProps,
          ...this.props
        }
      })
    }
    render() {
      const { allProps = {} } = this.state
      return <WrappedComponent {...allProps}/>
    }
  }
  return Connect
}
```

**connect的实现详解**

* mapStateToProps 告知Connect高阶组件从store中获取数据，然后将返回的状态（对象）传给被包装的组件WrappedComponent
* mapDispatchToProps 同理，告知Connect高阶组件从store获取dispatch，也就是methods, 通过props的形式传给被包装的组件
* WrappedComponent 被包装的组件，也就是PureComponent（DumbComponent）

看下mapStateToProps和mapDispatchToProps的使用

```javascript
const mapStateToProps = (state) => {
  return {
    themeColor: state.themeColor
  }
}
const mapDispatchToProps = (dispatch) => {
  return {
    changeColor: (color) => {
      dispatch(changeThemeColor(color))
    }
  }
}
```

结果会将themeColor和changeColor当做PureComponent的属性传入，PureComponent内部就会出现外部传入的状态

### React-redux 中Provider的实现

之前所有子组件的关于context的冗余和可复用问题通过Connect高阶组件得到了解决，但是关于context的另一个问题，父组件中定义全局context的逻辑是被污染的，因为要将context从所有的业务组件中清理出去。 因此产生了Provider组件，它是一个容器组件。

先来看下它的使用：

```javascript
import React, { Component } from 'react';
import {store} from './store'
import Provider from './provider'
// 将Header分成smart component 和 dumb component
import Header from './containers/Header'
import Content from './content'

export class ThemeIndex extends Component {
  render() {
    return (
      <div>
        <Provider store={store}>
          <Header headerProps={'headerhhh'}/>
          <Content/>
        </Provider>
      </div>
    );
  }
}
export default ThemeIndex;
```

**Provider是根组件**

Provider包含了所有的组件树，这样在其中处理context再好不过了。

**Provider高阶组件的实现**

```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types'

export class Provider extends Component {
  // 验证props的类型
  static propTypes = {
    store: PropTypes.object,
    children: PropTypes.any
  }
  // 验证context的类型 - 必须
  static childContextTypes = {
    store: PropTypes.object
  }
  // 定义context的值
  getChildContext() {
    return {
      store: this.props.store
    }
  }
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    )
  }
}
export default Provider;
```

### React-redux 中的Smart、Dumb

**Smart和Dumb** Smart 顾名思义，聪明的意思; Dumb，傻傻的意思；在我们的组件中，想要一个组件达到极强的复用性，必须使用Dumb组件，也就是Pure组件，但是如果组件不能做成一个Pure组件该怎么办呢？ 因为上述使用了Connect高阶组件，它可以完美的实现，但是pure组件和处理复杂业务的Connect高阶组件在一起，因此，可以将这两种不同的业务分开，这样，更近一步的增强了Dumb组件的复用性。

components文件专门用来存放一些Dumb组件，containers文件专门用来存放专门包装Dumb的组件，这也是一种工程化的习惯。

接下来看下具体的实现，就依据这次的实例中的Header组件

**Dumb component**

```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types'
export class Header extends Component {
  static propTypes = {
    themeColor: PropTypes.string
  }
  render() {
    const { themeColor } = this.props
    return (
      <div>
        <h1 style={{ color: themeColor }}>React-redux themeSwitch header</h1>
      </div>
    );
  }
}
export default Header;
```

**Smart component**

```javascript
import { connect } from '../connect'
import Header from '../components/Header'

const mapStateToProps = (state) => {
  return {
    themeColor: state.themeColor,
  }
}

export default connect(mapStateToProps)(Header);
```

**至此，react-redux 功能点以及大致的细节点完成了， 下节将所有功能串联起来，实现一个完整的更改主题颜色的例子**

### React-redux 实现完整Demo

#### 创建父组件Index

```javascript
import React, { Component } from 'react';
import {store} from './store'
import Provider from './provider'
// 将Header分成smart component 和 dumb component
import Header from './containers/Header'
import Content from './content'

export class ThemeIndex extends Component {
  render() {
    return (
      <div>
        <Provider store={store}>
          <Header headerProps={'headerhhh'}/>
          <Content/>
        </Provider>
      </div>
    );
  }
}
export default ThemeIndex;
```

#### 创建Header和Content

**Dumb Component**

1. Header Pure Component

   \`\`\`js

   // components/Header

   import React, { Component } from 'react';

   import PropTypes from 'prop-types'

export class Header extends Component { static propTypes = { themeColor: PropTypes.string }

render\(\) { const { themeColor } = this.props return \( React-redux themeSwitch header \); } }

export default Header;

```text
2. Content Pure Component
```js
import React, { Component } from 'react';
import PropTypes from 'prop-types'
import { connect } from './connect'
import Theme from './theme'
export class Content extends Component {
  static propsTypes = {
    themeColor: PropTypes.string
  }
  render() {
    const { themeColor } = this.props
    return (
      <div>
        <p style={{ color: themeColor}}>react-redux themeswitch</p>
        <Theme/>
      </div>
    );
  }
}
```

**Smart Components** 1. Header Smart Component

```javascript
// containers/Header
import { connect } from '../connect'
import Header from '../components/Header'
const mapStateToProps = (state, props) => {
  return {
    themeColor: state.themeColor,
    ...props
  }
}
export default connect(mapStateToProps)(Header);
```

1. Content Smart Component

```javascript
// containers/Content
import { connect } from '../connect'
import Content from '../components/Content'
const mapStateToProps = (state, props) => {
  return {
    themeColor: state.themeColor,
    ...props
  }
}
export default connect(mapStateToProps)(Content);
```

#### 创建Theme组件

```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types'
import { connect } from './connect'

export class Theme extends Component {
  static propTypes = {
    themeColor: PropTypes.string,
    onSwitchColor: PropTypes.func
  }

  handleThemeColor(color) {
    let { onSwitchColor } = this.props;
    if(onSwitchColor) {
      onSwitchColor(color)
    }
  }

  render() {
    const { themeColor } = this.props
    return (
      <div >
        <button style={{ color: themeColor }} onClick={this.handleThemeColor.bind(this, 'red')}>Red</button>
        <button style={{ color: themeColor }} onClick={this.handleThemeColor.bind(this, 'blue')}>Blue</button>
      </div>
    );
  }
}

const mapStateToProps = (state, props) => {
  return {
    themeColor: state.themeColor,
    ...props
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onSwitchColor: (color) => {
      dispatch({ type: "CHANGE_COLOR", themeColor: color })
    }
  }
}

Theme = connect(mapStateToProps, mapDispatchToProps)(Theme)
export default Theme;
```

### React-redux 实现总结

react-redux 其实就是context结合了redux 变成了react专有的状态管理机制。

**实现思路：**

1. 首先实现了createStore, 接收了函数reducer为参数，其功能包含了获取状态（getState）、订阅监听（subscribe）、修改状态（dispatch）
2. 紧接着实现了reducer, 它是一个纯函数，接收state和action为参数，初始化state，并根据action.type 来修改state
3. 之后为了解决子组件中context代码冗余问题，提出了高阶组件Connect，是专门将context和PureComponent连接了起来
4. 最后为了将context从业务组件中抽离，提出了Provider组件，专门处理有关context给到store的操作

react-redux大致的实现思路是这样，有关这个完整的例子，请看[这里](https://github.com/Wangbaoqi/react-nuggets/tree/master/src/reactStudy/redux)

