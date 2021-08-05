# Fiber 数据结构

### `FiberNode` 构造函数

```typescript
// FiberNode类型定义 react-reconciler/src/ReactInternalTypes.js

function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  // 标记不同组件类型 所有的枚举看Fiber tag type
  this.tag = tag;
  // ReactElement 中的key 
  this.key = key;
  // ReactElement.type，也就是我们调用createElement的第一个参数
  this.elementType = null;
  // 异步组件resolved之后返回的内容，一般是function或者class
  this.type = null;
  // 跟当前Fiber相关本地状态（比如浏览器环境就是DOM节点）
  this.stateNode = null;

  // Fiber
  // 指向其在Fiber节点树中的parent，用来在处理完这个节点之后向上返回
  this.return = null;
  // 单链表树结构
  // 指向自己的第一个子节点
  this.child = null;
  // 指向自己的兄弟节点
  // 兄弟节点的return指向同一个父节点
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  // 新的变动带来的新的props
  this.pendingProps = pendingProps;
  // 上一次渲染完成之后的props
  this.memoizedProps = null;
  // 该Fiber对应的组件产生的Update会存放在这个队列里面
  this.updateQueue = null;
  // 上一次渲染的时候的state
  this.memoizedState = null;
  
  this.dependencies = null;

  // 用来描述当前Fiber和他子树的`Bitfield`
  // 共存的模式表示这个子树是否默认是异步渲染的
  // Fiber被创建的时候他会继承父Fiber
  // 其他的标识也可以在创建的时候被设置
  // 但是在创建之后不应该再被修改，特别是他的子Fiber创建之前
  this.mode = mode;

  // Effects
  // 用来记录Side Effect
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;

  // 泳道 代替了之前的 expirationTime
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 在Fiber树更新的过程中，每个Fiber都会有一个跟其对应的Fiber
  // 我们称他为`current <==> workInProgress`
  // 在渲染完成之后他们会交换位置
  this.alternate = null;

  if (enableProfilerTimer) {
    // Note: The following is done to avoid a v8 performance cliff.
    //
    // Initializing the fields below to smis and later updating them with
    // double values will cause Fibers to end up having separate shapes.
    // This behavior/bug has something to do with Object.preventExtension().
    // Fortunately this only impacts DEV builds.
    // Unfortunately it makes React unusably slow for some applications.
    // To work around this, initialize the fields below with doubles.
    //
    // Learn more about this here:
    // https://github.com/facebook/react/issues/14365
    // https://bugs.chromium.org/p/v8/issues/detail?id=8538
    this.actualDuration = Number.NaN;
    this.actualStartTime = Number.NaN;
    this.selfBaseDuration = Number.NaN;
    this.treeBaseDuration = Number.NaN;

    // It's okay to replace the initial doubles with smis after initialization.
    // This won't trigger the performance cliff mentioned above,
    // and it simplifies other profiler code (including DevTools).
    this.actualDuration = 0;
    this.actualStartTime = -1;
    this.selfBaseDuration = 0;
    this.treeBaseDuration = 0;
  }

  
}
```

### Fiber TagType

Fiber Tag 代表不同组件的类型

```typescript
export const FunctionComponent = 0;
export const ClassComponent = 1;
export const IndeterminateComponent = 2; // Before we know whether it is function or class
export const HostRoot = 3; // Root of a host tree. Could be nested inside another node.
export const HostPortal = 4; // A subtree. Could be an entry point to a different renderer.
export const HostComponent = 5;
export const HostText = 6;
export const Fragment = 7;
export const Mode = 8;
export const ContextConsumer = 9;
export const ContextProvider = 10;
export const ForwardRef = 11;
export const Profiler = 12;
export const SuspenseComponent = 13;
export const MemoComponent = 14;
export const SimpleMemoComponent = 15;
export const LazyComponent = 16;
export const IncompleteClassComponent = 17;
export const DehydratedFragment = 18;
export const SuspenseListComponent = 19;
export const ScopeComponent = 21;
export const OffscreenComponent = 22;
export const LegacyHiddenComponent = 23;
export const CacheComponent = 24;
```

### Fiber ModeType

```typescript
export const NoMode = /*            */ 0b000000;
// TODO: Remove ConcurrentMode by reading from the root tag instead
export const ConcurrentMode = /*    */ 0b000001;
export const ProfileMode = /*       */ 0b000010;
export const DebugTracingMode = /*  */ 0b000100;
export const StrictLegacyMode = /*  */ 0b001000;
export const StrictEffectsMode = /* */ 0b010000;
```

