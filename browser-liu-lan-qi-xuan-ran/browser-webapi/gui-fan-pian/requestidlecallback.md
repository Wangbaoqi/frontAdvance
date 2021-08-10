# requestIdleCallback

`requestIdleCallback` 的出现是为了让[屏幕的刷新率](../../browser-xing-neng/xuan-ran-xing-neng/ping-mu-shua-xin.md)几乎保持在60HZ。这样页面在交互中不会卡顿。

`requestIdleCallback` 会在当前**帧结束之前的剩余时间里**或**用户不活动的时候调用**。这样意味着在不妨碍用户的前提完成任务。

### 使用 requestIdleCallback



