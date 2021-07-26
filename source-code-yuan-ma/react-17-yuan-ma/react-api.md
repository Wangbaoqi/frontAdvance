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

//避免为这些方法进行额外的原型跳转。 
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;
const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy())
pureComponentPrototype.constructor = PureComponent;
Object.assign(pureComponentPrototype, Component.prototype)
pureComponentPrototype.isPureReactComponent = true;
```

Component和pureComponent的定义除了pureComponent有`isPureComponent` 属性，其他基本一样。

正是这个属性，在组件的生命周期钩子函数中做个区分。

```javascript
// 
function checkShouldComponentUpdate(
  workInProgress,
  ctor,
  oldProps,
  newProps,
  oldState,
  newState,
  nextContext,
) {
  const instance = workInProgress.stateNode;
  if (typeof instance.shouldComponentUpdate === 'function') {
    const shouldUpdate = instance.shouldComponentUpdate(
      newProps,
      newState,
      nextContext,
    );
    return shouldUpdate;
  }

  // * component 和 pureComponent 之间的区别
  if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    return (
      !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
    );
  }
  return true;
}
```

可以看到，React.Component 并未实现 `shouldComponentUpdate`  ，而是直接使用了子类中定义的 `shouldComponentUpdate` ，而React.pureComponent 在没有自定义`shouldComponentUpdate`的基础上实现了其功能，shallowEqual 对新老state 和 新老props 进行了浅对比。

因此，当state和props是简单的数据类型的时候，才使用 React.pureComponent 。如果在纯组件中使用复杂结构时，更新用 `forceUpdate` 来确保数据对比的正确性。

```javascript
// pureComponent Test
class PureComp extends PureComponent<Props,State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      c: {
        maps: {
        name: 'nate'
      }
    }
  }
  changeName = () => {
    const { c = {} } = this.state
    c.maps.name = 'baoqi'
    // 这里setState 会失效 shouldComponentUpdate 中使用了浅比较
    // 使用 forceUpdate 会跳过 shouldComponentUpdate 直接调用render
    this.setState({
      c
    })
  }
  render() {
    const { c = {} } = this.state
    return (
      <div>
        <p>name: {c.maps.name}</p>
        <button onClick={this.changeName}>CHANGE NAME</button>
      </div>
    )
  }
}
```

### React.memo

React.memo 的使用详情可以到 [「React Family - React API」](../../frame-react-and-vue/react-family/react-jsx.md#react-memo)查看。这里着重看下源码

```javascript
function memo<Props>(
  type: React$ElementType,
  compare?: (oldProps: Props, newProps: Props) => boolean,
) {
  const elementType = {
    $$typeof: REACT_MEMO_TYPE,
    type,
    compare: compare === undefined ? null : compare,
  };
  return elementType;
}
```

可以看到，React.memo 接收了 `reactElement` 和 `compare`（对比新老自定义方法），返回了标识 **REACT\_MEMO\_TYPE** _的元素。_

跟普通组件的差异是在调度阶段。

```javascript
// react/packages/react-reconciler/src/ReactFiberBeginWork.old.js
function updateSimpleMemoComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: any,
  nextProps: any,
  updateLanes: Lanes,
  renderLanes: Lanes,
): null | Fiber {
  if (current !== null) {
    const prevProps = current.memoizedProps;
    // 新老props 进行浅比较 
    if (
      shallowEqual(prevProps, nextProps) &&
      current.ref === workInProgress.ref &&
    ) {
      didReceiveUpdate = false;
      if (!includesSomeLane(renderLanes, updateLanes)) {
        workInProgress.lanes = current.lanes;
        return bailoutOnAlreadyFinishedWork(
          current,
          workInProgress,
          renderLanes,
        );
      } else if ((current.flags & ForceUpdateForLegacySuspense) !== NoFlags) {
        didReceiveUpdate = true;
      }
    }
  }
  return updateFunctionComponent(
    current,
    workInProgress,
    Component,
    nextProps,
    renderLanes,
  );
}
```

当新老`props`浅比较`shallowEqual`的结果相同时，直接执行`bailoutOnAlreadyFinishedWork`，那么当前组件就会被记忆化，因此不会进行diff操作等。

### React.createRef

React.createRef的源码很简单，返回带有`current`属性的对象。

```javascript
function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  return refObject;
}
```

在[「React Family - React API  React.createRef」](../../frame-react-and-vue/react-family/react-jsx.md#react-createref)中提到了Refs的执行时机

ref在定义之后，是直接绑定到组件的实例上的，这个绑定的过程是在组件挂载的时候，那什么时候将DOM元素或者React元素绑定到Ref上呢？

后面在说到React任务调度的时候，会说到**Render 阶段**和**Commit 阶段**，ref的结果赋值是在**Commit阶段**，

这里的ref只针对**非字符ref，也就是通过 createRef 或者 回调 Ref 创建的 Ref。**

**非字符Ref是在Render阶段处理，细一点就是在 Diff 操作的时候处理，这里涉及到源码，具体的整个流程后面在React调度中会具体阐述。**

#### **处理字符Ref 源码** 

```javascript
// react/packages/react-reconciler/src/ReactChildFiber.old.js
function coerceRef(
  returnFiber: Fiber,
  current: Fiber | null,
  element: ReactElement,
) {
  const mixedRef = element.ref;
  if (
    mixedRef !== null &&
    typeof mixedRef !== 'function' &&
    typeof mixedRef !== 'object'
  ) {
    if (element._owner) {
      const owner: ?Fiber = (element._owner: any);
      let inst;
      if (owner) {
        const ownerFiber = ((owner: any): Fiber);
        inst = ownerFiber.stateNode;
      }
      const stringRef = '' + mixedRef;
      // Check if previous string ref matches new string ref
      if (
        current !== null &&
        current.ref !== null &&
        typeof current.ref === 'function' &&
        current.ref._stringRef === stringRef
      ) {
        return current.ref;
      }
      const ref = function(value) {
        let refs = inst.refs;
        if (refs === emptyRefsObject) {
          // This is a lazy pooled frozen object, so we need to initialize.
          refs = inst.refs = {};
        }
        if (value === null) {
          delete refs[stringRef];
        } else {
          refs[stringRef] = value;
        }
      };
      ref._stringRef = stringRef;
      return ref;
    } else {
      // error...
    }
  }
  return mixedRef;
}
```

#### 处理非字符Ref

非字符Ref是在Commit阶段处理的，下面是具体的源码

```javascript
// react/packages/react-reconciler/src/ReactFiberCommitWork.old.js
function commitAttachRef(finishedWork: Fiber) {
  const ref = finishedWork.ref;
  if (ref !== null) {
    const instance = finishedWork.stateNode;
    let instanceToUse;
    switch (finishedWork.tag) {
      case HostComponent:
        // 获取DOM几点
        instanceToUse = getPublicInstance(instance);
        break;
      default:
        instanceToUse = instance;
    }
    // Moved outside to ensure DCE works with this flag
    if (enableScopeAPI && finishedWork.tag === ScopeComponent) {
      instanceToUse = instance;
    }
    // ref 是callback 的情况
    if (typeof ref === 'function') {
      if (
        enableProfilerTimer &&
        enableProfilerCommitHooks &&
        finishedWork.mode & ProfileMode
      ) {
        try {
          startLayoutEffectTimer();
          ref(instanceToUse);
        } finally {
          recordLayoutEffectDuration(finishedWork);
        }
      } else {
        ref(instanceToUse);
      }
    } else {
      ref.current = instanceToUse;
    }
  }
}
```

`commitAttachRef` 接收带有Ref的组件（这里是Fiber节点）

如果是通过`createRef`创建的Ref，获取其DOM节点，赋值给`current`属性。

如果是通过`callback` 创建的Ref，执行回调函数，将DOM节点作为回调函数的参数。

#### 组件卸载时卸载Ref

在组件卸载是，ref 同样被卸载，`createRef` 和 `callback` 都会传入`null` 来卸载。

```javascript
// react/packages/react-reconciler/src/ReactFiberCommitWork.old.js
function commitDetachRef(current: Fiber) {
  const currentRef = current.ref;
  if (currentRef !== null) {
    if (typeof currentRef === 'function') {
      if (
        enableProfilerTimer &&
        enableProfilerCommitHooks &&
        current.mode & ProfileMode
      ) {
        try {
          startLayoutEffectTimer();
          currentRef(null);
        } finally {
          recordLayoutEffectDuration(current);
        }
      } else {
        currentRef(null);
      }
    } else {
      currentRef.current = null;
    }
  }
}
```

### React.forwardRef

React.forwardRef 接收一个函数为参数，返回一个REACT\_FORWARD\_REF\_TYPE类型的对象。 回调函数接收 props 和 ref ，

```javascript
function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
  const elementType = {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render,
  };
  return elementType;
}
```

在React任务调度过程中，如果解析到了类型为REACT\_FORWARD\_REF\_TYPE的组件时候就会执行转发Refs的逻辑，进入到`updateForwardRef`  函数中。接收参数 Component 就是React.forwardRef 创建的元素。 

接下来获取到了当前Fiber节点上的ref，此时为空的ref。

```javascript
// beginWork 
// react/packages/react-reconciler/src/ReactFiberBeginWork.old.js
function updateForwardRef(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: any,
  nextProps: any,
  renderLanes: Lanes,
) {
  
  const render = Component.render;
  const ref = workInProgress.ref;

  let nextChildren;
  prepareToReadContext(workInProgress, renderLanes);
  if (__DEV__) {
    // ...
  } else {
    nextChildren = renderWithHooks(
      current,
      workInProgress,
      render,
      nextProps,
      ref,
      renderLanes,
    );
  }

  if (current !== null && !didReceiveUpdate) {
    bailoutHooks(current, workInProgress, renderLanes);
    return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
  }

  // React DevTools reads this flag.
  workInProgress.flags |= PerformedWork;
  reconcileChildren(current, workInProgress, nextChildren, renderLanes);
  return workInProgress.child;
}
```

接下来调用 `renderWithHooks` ，执行`render`方法，返回的是forwardRef中回调函数返回的组件，作为当前fiber节点的子fiber节点。

```javascript
function renderWithHooks<Props, SecondArg>(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: (p: Props, arg: SecondArg) => any,
  props: Props,
  secondArg: SecondArg,
  nextRenderLanes: Lanes,
): any {
  // ...
  // props secondArg = ref
  let children = Component(props, secondArg);
  // ...
  return children;
}
```

最后ref的赋值也是在Commit阶段，跟createRef是一致的。

### React.lazy

React.lazy 源码定义直接返回一个带有类型的REACT\_LAZY\_TYPE的对象

```javascript
function lazy<T>(
  ctor: () => Thenable<{default: T, ...}>,
): LazyComponent<T, Payload<T>> {
  const payload: Payload<T> = {
    _status: -1,
    _result: ctor,
  };

  const lazyType: LazyComponent<T, Payload<T>> = {
    $$typeof: REACT_LAZY_TYPE,
    _payload: payload,
    _init: lazyInitializer,
  };

  return lazyType;
}
```

接收函数为参数，返回一个具有**Thenable**接口的`Promise`

```javascript
function lazyInitializer<T>(payload: Payload<T>): T {
  if (payload._status === Uninitialized) {
    const ctor = payload._result;
    const thenable = ctor();
    // Transition to the next state.
    const pending: PendingPayload = (payload: any);
    pending._status = Pending;
    pending._result = thenable;
    thenable.then(
      moduleObject => {
        if (payload._status === Pending) {
          const defaultExport = moduleObject.default;
          // Transition to the next state.
          const resolved: ResolvedPayload<T> = (payload: any);
          resolved._status = Resolved;
          resolved._result = defaultExport;
        }
      },
      error => {
        if (payload._status === Pending) {
          // Transition to the next state.
          const rejected: RejectedPayload = (payload: any);
          rejected._status = Rejected;
          rejected._result = error;
        }
      },
    );
  }
  if (payload._status === Resolved) {
    return payload._result;
  } else {
    throw payload._result;
  }
}
```



### React.Suspense 





```javascript

function updateSuspenseComponent(current, workInProgress, renderLanes) {
  const nextProps = workInProgress.pendingProps;
  let suspenseContext: SuspenseContext = suspenseStackCursor.current;
  let showFallback = false;
  const didSuspend = (workInProgress.flags & DidCapture) !== NoFlags;

  if (
    didSuspend ||
    shouldRemainOnFallback(
      suspenseContext,
      current,
      workInProgress,
      renderLanes,
    )
  ) {
    // Something in this boundary's subtree already suspended. Switch to
    // rendering the fallback children.
    showFallback = true;
    workInProgress.flags &= ~DidCapture;
  } else {
    // Attempting the main content
    if (
      current === null ||
      (current.memoizedState: null | SuspenseState) !== null
    ) {
      if (
        nextProps.fallback !== undefined &&
        nextProps.unstable_avoidThisFallback !== true
      ) {
        suspenseContext = addSubtreeSuspenseContext(
          suspenseContext,
          InvisibleParentSuspenseContext,
        );
      }
    }
  }
  suspenseContext = setDefaultShallowSuspenseContext(suspenseContext);
  pushSuspenseContext(workInProgress, suspenseContext);

  if (current === null) {

    const nextPrimaryChildren = nextProps.children;
    const nextFallbackChildren = nextProps.fallback;
    if (showFallback) {
      const fallbackFragment = mountSuspenseFallbackChildren(
        workInProgress,
        nextPrimaryChildren,
        nextFallbackChildren,
        renderLanes,
      );
      const primaryChildFragment: Fiber = (workInProgress.child: any);
      primaryChildFragment.memoizedState = mountSuspenseOffscreenState(
        renderLanes,
      );
      workInProgress.memoizedState = SUSPENDED_MARKER;
      return fallbackFragment;
    } else if (typeof nextProps.unstable_expectedLoadTime === 'number') {
      // This is a CPU-bound tree. Skip this tree and show a placeholder to
      // unblock the surrounding content. Then immediately retry after the
      // initial commit.
      const fallbackFragment = mountSuspenseFallbackChildren(
        workInProgress,
        nextPrimaryChildren,
        nextFallbackChildren,
        renderLanes,
      );
      const primaryChildFragment: Fiber = (workInProgress.child: any);
      primaryChildFragment.memoizedState = mountSuspenseOffscreenState(
        renderLanes,
      );
      workInProgress.memoizedState = SUSPENDED_MARKER;

      workInProgress.lanes = SomeRetryLane;
      return fallbackFragment;
    } else {
      
      return mountSuspensePrimaryChildren(
        workInProgress,
        nextPrimaryChildren,
        renderLanes,
      );
    }
  } else {
    // This is an update.

    // If the current fiber has a SuspenseState, that means it's already showing
    // a fallback.
    const prevState: null | SuspenseState = current.memoizedState;
    if (prevState !== null) {
      // The current tree is already showing a fallback

      // Special path for hydration
      if (enableSuspenseServerRenderer) {
        // ...
      }

      if (showFallback) {
        const nextFallbackChildren = nextProps.fallback;
        const nextPrimaryChildren = nextProps.children;
        const fallbackChildFragment = updateSuspenseFallbackChildren(
          current,
          workInProgress,
          nextPrimaryChildren,
          nextFallbackChildren,
          renderLanes,
        );
        const primaryChildFragment: Fiber = (workInProgress.child: any);
        const prevOffscreenState: OffscreenState | null = (current.child: any)
          .memoizedState;
        primaryChildFragment.memoizedState =
          prevOffscreenState === null
            ? mountSuspenseOffscreenState(renderLanes)
            : updateSuspenseOffscreenState(prevOffscreenState, renderLanes);
        primaryChildFragment.childLanes = getRemainingWorkInPrimaryTree(
          current,
          renderLanes,
        );
        workInProgress.memoizedState = SUSPENDED_MARKER;
        return fallbackChildFragment;
      } else {
        const nextPrimaryChildren = nextProps.children;
        const primaryChildFragment = updateSuspensePrimaryChildren(
          current,
          workInProgress,
          nextPrimaryChildren,
          renderLanes,
        );
        workInProgress.memoizedState = null;
        return primaryChildFragment;
      }
    } else {
      // The current tree is not already showing a fallback.
      if (showFallback) {
        // Timed out.
        const nextFallbackChildren = nextProps.fallback;
        const nextPrimaryChildren = nextProps.children;
        const fallbackChildFragment = updateSuspenseFallbackChildren(
          current,
          workInProgress,
          nextPrimaryChildren,
          nextFallbackChildren,
          renderLanes,
        );
        const primaryChildFragment: Fiber = (workInProgress.child: any);
        const prevOffscreenState: OffscreenState | null = (current.child: any)
          .memoizedState;
        primaryChildFragment.memoizedState =
          prevOffscreenState === null
            ? mountSuspenseOffscreenState(renderLanes)
            : updateSuspenseOffscreenState(prevOffscreenState, renderLanes);
        primaryChildFragment.childLanes = getRemainingWorkInPrimaryTree(
          current,
          renderLanes,
        );
        // Skip the primary children, and continue working on the
        // fallback children.
        workInProgress.memoizedState = SUSPENDED_MARKER;
        return fallbackChildFragment;
      } else {
        // Still haven't timed out. Continue rendering the children, like we
        // normally do.
        const nextPrimaryChildren = nextProps.children;
        const primaryChildFragment = updateSuspensePrimaryChildren(
          current,
          workInProgress,
          nextPrimaryChildren,
          renderLanes,
        );
        workInProgress.memoizedState = null;
        return primaryChildFragment;
      }
    }
  }
}
```





```javascript
function mountSuspensePrimaryChildren(
  workInProgress,
  primaryChildren,
  renderLanes,
) {
  const mode = workInProgress.mode;
  const primaryChildProps: OffscreenProps = {
    mode: 'visible',
    children: primaryChildren,
  };
  const primaryChildFragment = createFiberFromOffscreen(
    primaryChildProps,
    mode,
    renderLanes,
    null,
  );
  primaryChildFragment.return = workInProgress;
  workInProgress.child = primaryChildFragment;
  return primaryChildFragment;
}
```

