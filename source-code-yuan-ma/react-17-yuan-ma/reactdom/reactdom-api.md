# ReactDOM API

在**ReactDOM**中，使用最多的无疑是 `render` API，这也是整个应用第一次渲染的入口方法，下面逐一分析下这个`render`方法中做了什么操作。

在React源码中，ReactDOM是单独的一个包，也是很重要的一个包。

![](../../../.gitbook/assets/reactdom-api.png)

### ReactDOM.render

```typescript
// react-dom/src/client/ReactDOMLegacy.js
function render(
  element: React$Element<any>,
  container: Container,
  callback: ?Function,
) { 
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}
```

可以看到 render 方法接收的参数为 `element(ReactElement)` ，`container(挂载的容器节点)` ，`callback`  ，之后调用了 `legacyRenderSubtreeIntoContainer` 方法，该方法是创建 reactRoot的入口方法，也是整个应用更新的入口方法。

```typescript
// react-dom/src/client/ReactDOMLegacy.js
// 展示了核心代码
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  container: Container,
  forceHydrate: boolean,
  callback: ?Function,
) {
  
  let root = container._reactRootContainer;
  let fiberRoot: FiberRoot;
  // 第一次渲染
  if (!root) {
    // Initial mount
    // 得到 root = new ReactDOMLegacyRoot 实例
    // 创建 ReactRoot 开始 
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    
    fiberRoot = root._internalRoot;
    
    // Initial mount should not be batched. 
    // 未分批更新
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // Update
    updateContainer(children, fiberRoot, parentComponent, callback);
  }

  // TODO 跟16.6版本区别 
  // return DOMRenderer.getPublicRootInstance(root._internalRoot);

  return getPublicRootInstance(fiberRoot);
}
```

可以看到，第一次渲染的时候，调用了 `legacyCreateRootFromDOMContainer` 方法，返回了`root` ，整个`root` 就是 **ReactRoot，ReactRoot是一个数据结构，记录了整个根应用的信息。其实例属性记录了FiberRoot的信息，FiberRoot 后面会涉及到。**

接下来就开始创建**ReactRoot**了

```typescript
// react-dom/src/client/ReactDOMLegacy.js
function legacyCreateRootFromDOMContainer(
  container: Container,
  forceHydrate: boolean,
): RootType {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  // First clear any existing content.

  // 第一次渲染 并且不是服务端渲染 清空容器中节点
  if (!shouldHydrate) {
    let warned = false;
    let rootSibling;
    while ((rootSibling = container.lastChild)) {
      container.removeChild(rootSibling);
    }
  }

  // 创建ReactRoot节点
  return createLegacyRoot(
    container,
    shouldHydrate
      ? {
          hydrate: true,
        }
      : undefined,
  );
}
```

#### Create ReactRoot

 创建 ReactRoot

```typescript
// react-dom/src/client/ReactDOMRoot.js
function createLegacyRoot(
  container: Container,
  options?: RootOptions,
): RootType {
  return new ReactDOMLegacyRoot(container, options);
}
```

ReactDOMLegacyRoot 构造方法

```typescript
// react-dom/src/client/ReactDOMRoot.js
function ReactDOMLegacyRoot(container: Container, options: void | RootOptions) {
  // LegacyRoot: 0 options: undefined
  // createRootImpl ReateRoot实现
  this._internalRoot = createRootImpl(container, LegacyRoot, options);
}
// 实现render 方法
ReactDOMLegacyRoot.prototype.render = function(
  children: ReactNodeList,
): void {
  const root = this._internalRoot;
  updateContainer(children, root, null, null);
};
// 实现unmount 方法
ReactDOMLegacyRoot.prototype.unmount = function(): void {
  const root = this._internalRoot;
  const container = root.containerInfo;
  updateContainer(null, root, null, () => {
    unmarkContainerAsRoot(container);
  });
};
```

`createRootImpl` 是`FiberRoot`实现，注意这里的 `LegacyRoot = 0`  与之类似的还有一个 `ConcurrentRoot = 1` 

```typescript
// react-dom/src/client/ReactDOMRoot.js
function createRootImpl(
  container: Container,
  tag: RootTag,
  options: void | RootOptions,
) {

  // createContainer(el, 0, null, null, null)
  // 创建FiberRoot入口方法
  const root = createContainer(
    container,
    tag,
    hydrate,
    hydrationCallbacks,
    strictModeLevelOverride,
  );
  
  markContainerAsRoot(root.current, container);

  return root;
}
```

#### Create FiberRoot



```typescript
// react-reconciler/ReactFiberRoot.new.js
function createContainer(
  containerInfo: Container,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
  strictModeLevelOverride: null | number,
): OpaqueRoot {
  return createFiberRoot(
    containerInfo,
    tag,
    hydrate,
    hydrationCallbacks,
    strictModeLevelOverride,
  );
}

// FiberRoot 实现方法
function createFiberRoot(
  containerInfo: any,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
  strictModeLevelOverride: null | number,
): FiberRoot {
  // containerInfo: el tag: 0 strictModeLevelOverride: null
  
  // 创建FiberRoot实例
  const root: FiberRoot = (new FiberRootNode(containerInfo, tag, hydrate): any);
  

  // Cyclic construction. This cheats the type system right now because
  // stateNode is any.
  
  const uninitializedFiber = createHostRootFiber(tag, strictModeLevelOverride);
  // FiberRoot 对应的Fiber对象
  root.current = uninitializedFiber;
  // 跟当前Fiber相关本地状态（比如浏览器环境就是DOM节点）
  uninitializedFiber.stateNode = root;
  // 初始化更新队列
  initializeUpdateQueue(uninitializedFiber);

  return root;
}

```

关于 FiberRootNode 的数据结构[到这里](../react-chuang-jian-geng-xin/fiberroot-shu-ju-jie-gou.md)，FiberNode数据结构在[这里](../react-chuang-jian-geng-xin/fiber-shu-ju-jie-gou.md)。

紧接着创建`RootFiber` ，其实也是`Fiber`对象，不过他的 `tag` 属性代表的是根组件类型。

最后初始化更新队列，返回`FiberRoot` 实例。

#### InitUpdateQueue

初始化更新队列中将 `UpdateQueue` 结构赋给了Fiber对象上的`updateQueue`属性

```typescript
function initializeUpdateQueue<State>(fiber: Fiber): void {
  const queue: UpdateQueue<State> = {
    // 每次操作完更新之后的state
    baseState: fiber.memoizedState,
    // 队列中第一个Update
    firstBaseUpdate: null,
    // 队列中最后一个Update    
    lastBaseUpdate: null,
    shared: {
      pending: null,
      interleaved: null,
      lanes: NoLanes,
    },
    effects: null,
  };
  fiber.updateQueue = queue;
}
```

此时，所有初始化好了所需要的数据结构，接下来返回到下面代码，进入更新阶段。

```javascript
root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
    container,
    forceHydrate,
);
    
fiberRoot = root._internalRoot;
    
// Initial mount should not be batched. 
// 未分批更新
unbatchedUpdates(() => {
  updateContainer(children, fiberRoot, parentComponent, callback);
});
```



