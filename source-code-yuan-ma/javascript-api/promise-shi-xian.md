# Promise 实现

Promise 的实现要遵循 [Promise/A+](https://promisesaplus.com/) 规范，其实现过程也可以进一步掌握 Promise 的原理。该规范主要是实现了 **thenable** 的接口。

之前[期约和异步函数](../../javascript-advance/javascript-jin-jie/sync-promise.md)详细解释了Promise，接下里通过代码的方式来理解比较抽象的概念。

### 术语

**thenable** 是定义 then 方法的函数或者对象。

**exception** 是使用 **throw** 抛出来的值。

**reason** 是用来表示被拒绝的原因。

### Promise Status

promise 有三种状态（pending、fulfilled 以及 rejected）。

当promise状态处于待定（pending）状态时，可以转换为已解决（fulfilled）或者 已拒绝（rejected）。

当promise状态处于已解决（fulfilled）状态时，不可以再转换为其他状态，此时必须有一个值，且该值不可改变。

当promise状态处于已拒绝（rejected）状态时，不可以再转换为其他状态，此时必须有一个值，且该值不可改变。

**\*** 这里, “不可改变” 意味者恒等 \(即通过 === 判断\), 而不意味者更深层次的不可变。

```javascript
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED =  'rejected';
```

### Promise Constructor

**Promise** 是一个构造函数，接收一个执行器（**excutor**）函数，该执行器函数接收 resolve （已解决）和reject（已拒绝）函数，用来改变promise状态。

```javascript
function PromiseN(excutor) {
  const self = this;
  // 当前状态
  this.status = PENDING;
  // 已解决值
  this.value = null;
  // 已拒绝原因
  this.reason = null;
  // onFulFilled 和 onRejected 用来保存 then 中的回调
  this.onFulFilled = [];
  this.onRejected = [];
  
  // 初始化 resolve 和 reject 函数
  // ...
  
  try {
    excutor(resolve, reject)
  }catch(e) {
    reject(e)
  }
}
```

上述初始化了Promise构造函数内部私有值

* **status**： 初始化当前状态 - 待定
* **value**：初始化已解决状态的值 - 默认为null
* **reason**： 初始化已拒绝状态的值 - 默认为null
* **onFulFilled**： 用来保存then中已解决状态的回调
* **onRejected**： 用来保存then中已拒绝状态的回调

之后执行**执行器函数，**传递 **resolve** 和 **reject** 函数用来异步调用，这里注意一点，**执行器函数中的同步代码是首先执行的**。接下来实现 **resolve** 和 **reject** 函数。

```javascript
// 状态落地为已解决函数
function resolve(val) {
  if(self.status === PENDING) {
    self.status = FULFILLED;
    self.value = val;
    self.onFulFilled.map(fn => fn());
  }
}
// 状态落地为已拒绝函数
function reject(reason) {
  if(self.status === PENDING) {
    self.status = REJECTED;
    self.reason = reason;
    self.onRejected.map(fn => fn());
  }
}
```

上述两个函数分别当状态落定为**已解决**或者**已拒绝**时执行。

规范规定，只有状态为待定时才可以改变为 **resolve** 或者 **reject**，因此在改变状态之前需要判断一下。之后将待定状态改变为对应落定状态，然后执行回调函数数组。

到此为止，简易的Promise构造函数就完成了，接下来测试一下

```javascript
const testPromise = new PromiseN((resolve, reject) => {
  resolve('testPromise')
})
// PromiseN{ status: 'fulFilled', onFulFilled: [], onRejected: [], value: 'testPromise' }

const promise = new Promise((resolve, reject) => {
  resolve('promise')
})
// Promise{ <fulfilled>: 'promise'}
```

可以看到，当执行**resolve 执行器**之后，状态更改为 **fulFIlled** ，已解决状态当前的值也更改了。对比原生Promise，其结果也是一致。

### Promise then



