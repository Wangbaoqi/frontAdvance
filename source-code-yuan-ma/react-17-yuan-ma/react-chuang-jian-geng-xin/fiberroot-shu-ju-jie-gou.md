# FiberRoot 数据结构



`FiberRoot` 构造函数

```typescript
// react-reconciler/src/ReactFiberRoot.old.js
function FiberRootNode(containerInfo, tag, hydrate) {
  // tag 有两种类型 LegacyRoot = 0 ConcurrentRoot = 1
  this.tag = tag;
  // 挂载的容器
  this.containerInfo = containerInfo;
  // 只有在持久更新中会用到，也就是不支持增量更新的平台，react-dom不会用到
  this.pendingChildren = null;
  // 当前应用对应的Fiber对象 是FiberRoot，其实是通过createHostRootFiber得到
  this.current = null;
  
  this.pingCache = null;
  // 在commit阶段置灰处理这个值对应的任务
  this.finishedWork = null;
  // 在任务被挂起的时候通过setTimeout设置的返回内容，
  // 用来下一次如果有新的任务挂起时清理还没触发的timeout.
  this.timeoutHandle = noTimeout;
  // 顶层context对象，只有主动调用 renderSubtreeIntoContainer 时才会有用
  this.context = null;
  this.pendingContext = null;
  // 用来确定第一次渲染的时候是否需要融合
  this.hydrate = hydrate;
  
  // Scheduler.scheduleCallback 返回的节点。 代表下一次渲染
  this.callbackNode = null;
  this.callbackPriority = NoLane;
  this.eventTimes = createLaneMap(NoLanes);
  this.expirationTimes = createLaneMap(NoTimestamp);

  this.pendingLanes = NoLanes;
  this.suspendedLanes = NoLanes;
  this.pingedLanes = NoLanes;
  this.expiredLanes = NoLanes;
  this.mutableReadLanes = NoLanes;
  this.finishedLanes = NoLanes;

  this.entangledLanes = NoLanes;
  this.entanglements = createLaneMap(NoLanes);

  if (enableCache) {
    this.pooledCache = null;
    this.pooledCacheLanes = NoLanes;
  }

  if (supportsHydration) {
    this.mutableSourceEagerHydrationData = null;
  }

  if (enableSuspenseCallback) {
    this.hydrationCallbacks = null;
  }

  if (enableProfilerTimer && enableProfilerCommitHooks) {
    this.effectDuration = 0;
    this.passiveEffectDuration = 0;
  }

  if (enableUpdaterTracking) {
    this.memoizedUpdaters = new Set();
    const pendingUpdatersLaneMap = (this.pendingUpdatersLaneMap = []);
    for (let i = 0; i < TotalLanes; i++) {
      pendingUpdatersLaneMap.push(new Set());
    }
  }
}
```





