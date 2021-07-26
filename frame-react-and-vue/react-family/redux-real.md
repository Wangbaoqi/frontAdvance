# redux - 状态管理



### 状态管理

一个比较复杂的项目如果没有约定性的随意修改组件的状态，是一件比较可怕的事，对于debug来讲，增加排查的成本，也会导致整个项目难以维护，代码的复用性基本上没有可用之处， 因此，redux 提出了这种统一管理状态的机制，如果要修改状态的话，必须大张旗鼓的，通过diapatch修改

先看个🌰:

```javascript
// 定义一个reducer
function reducer(state, action) {}

// 生成store
function createStore(reducer) {
  // ...
  return { getState, subscribe, dispatch }
}

const store = createStore(reducer)

// 订阅 监听数据变化重新渲染页面
store.subscribe(() => renderApp(store.getState()))

// 修改数据
store.dispatch(action)
```

**手动实现Redux实例**

**创建节点** 创建节点是为了在react项目中测试

```javascript
// 创建节点
function appendRoot() {
  let root = document.getElementById('root')
  let nodeTitle = document.createElement('div')
  let nodeContent = document.createElement('div')

  nodeTitle.id = 'time'
  nodeContent.id = 'content'

  root.appendChild(nodeTitle)
  root.appendChild(nodeContent)
}
```

页面显示节点

```markup
<div id='title'></div>
<div id='content'></div>
```

### state 页面状态

```javascript
// init state
const appState = {
  title: {
    text: 'react之手写Redux01',
    color: 'red'
  },
  content: {
    text: 'react之手写Redux02',
    color: 'blue'
  }
}
```

### render 将数据渲染到页面

在渲染页面中，暂时使用原生方式

```javascript
// 渲染ID为title的节点
function renderTitle(newState, oldState) {

  let nodeTitle = document.getElementById('title');
  nodeTitle.innerHtml = newState.text;
  nodeTitle.style.color = newState.color;
}

// 渲染ID为title的节点
function renderContent(newState, oldState) {

  let nodecontent = document.getElementById('content');
  nodecontent.innerHtml = newState.text;
  nodecontent.style.color = newState.color;
}

// 渲染所有页面
function renderApp(newState, oldState) {

  renderTitle(newState, oldState);
  renderContent(newState, oldState);
}
// 收口渲染
function render() {

  // render page
  renderApp(appState)
}
```

一个简单的页面初始化流程就弄好了，接下来通过简易手写redux来渲染页面

### reducer 初始化state

reducer 是专门处理state的func，初始化state和根据action的type来修改state，是一个纯函数

```javascript
// 定义一个reducer
function reducer(state, action) {
  // 初始化state 首次没有更新state
  if(!state) {
    return {
      title: {
        text: 'react之手写Redux01',
        color: 'red',
      },
      content: {
        text: 'react之手写Redux02',
        color: 'blue'
      }
    }
  }
  // 根据action type 来修改数据，注意，返回的是一个全新的对象，更新的数据是已经变动的数据，没有变动的数据两个对象指向的地址还是同一个 
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      return {
        ...state,
        title: {
          ...state.title,
          text: action.text
        }
      }
    case 'UPDATE_TITLE_COLOR':
      return {
        ...state,
        title: {
          ...state.title,
          color: action.color
        }
      }
    default:
      return false;
  }
}
```

### createStore 收集state 修改state

createStore 从文章开头的例子看出，接受reducer为入参， 返回了 getState\(获取state\)，subscribe\(订阅 修改state的function\)，dispatch\(通过传入的action的type来修改state\)\)

```javascript
function createStore(reducer) {
  let listeners = [];
  let state = null;

  const getState = () => state;
  const subscribe = (listener) => listeners.push(listener);
  // 修改state 
  const dispatch = (action) => {
    // 更新改动的数据 减少数据渲染次数
    state = reducer(state, action);
    // 执行订阅的更新state的function
    listeners.forEach(listener => listener())
  }

  // 首次初始化state
  dispatch({})

  return { getState, subscribe, dispatch }
}
```

**核心code已经完成，接下来看它的使用**

紧接着再次更新render方法

```javascript
// 收口渲染
function render() {
  // 获取store 
  const store = createStore(reducer)

  store.dispatch({ type: 'UPDATE_TITLE_TEXT', text: 'this is a redux case'})
  store.dispatch({ type: 'UPDATE_TITLE_COLOR', color: 'green'})

  // render page
  renderApp(appState)
}
```

