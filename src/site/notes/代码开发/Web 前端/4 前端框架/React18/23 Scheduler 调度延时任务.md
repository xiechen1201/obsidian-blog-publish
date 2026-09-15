---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/23 Scheduler 调度延时任务/","dg-note-properties":{}}
---

承接 [[代码开发/Web 前端/4 前端框架/React18/22 Scheduler 调度普通任务\|22 Scheduler 调度普通任务]] 中的 `unstable_scheduleCallback()` 函数，本文来看下 Scheduler 如何调度延时任务。

```js hl:15-37

// ...

/**
 *
 * @param {*} priorityLevel 优先级等级
 * @param {*} callback 具体要做的任务
 * @param {*} options { delay: number } 这是一个对象，该对象有 delay 属性，表示要延迟的时间
 * @returns
 */
function unstable_scheduleCallback(priorityLevel, callback, options) {

  // ...
  
  // 如果开始是时间 > 当前时间
  // 说明这是一个延时任务
  if (startTime > currentTime) {
    // This is a delayed task.
    newTask.sortIndex = startTime;
    // 将该任务推入到 timerQueue 的任务队列中
    push(timerQueue, newTask);
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // 进入此 if，说明 taskQueue 里面的任务已经执行完毕了
      // 并且从 timerQueue 里面取出一个最新的任务又是当前任务
      // All tasks are delayed, and this is the task with the earliest delay.

      // 下面的 if.. else 就是一个开关
      if (isHostTimeoutScheduled) {
        // Cancel an existing timeout.
        cancelHostTimeout();
      } else {
        isHostTimeoutScheduled = true;
      }
      // Schedule a timeout.
      // 如果是延时任务，调用 requestHostTimeout 把延时任务放入到 taskQueue 中
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    // 说明不是延时任务
    newTask.sortIndex = expirationTime; // 设置了 sortIndex 后，可以在任务队列里面进行一个排序
    // 推入到 taskQueue 任务队列
    push(taskQueue, newTask);
    if (enableProfiling) {
      markTaskStart(newTask, currentTime);
      newTask.isQueued = true;
    }
    // Schedule a host callback, if needed. If we're already performing work,
    // wait until the next time we yield.
    // 最终调用 requestHostCallback 进行任务的调度
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    }
  }

  // 向外部返回任务
  return newTask;
}
```

可以看到延时任务主要是执行 `requestHostTimeout()` 方法。

## requestHostTimeout

```js
// 实际上在浏览器环境就是 setTimeout
const localSetTimeout = typeof setTimeout === 'function' ? setTimeout : null;

/**
 * 
 * @param {*} callback 就是传入的 handleTimeout
 * @param {*} ms 延时的时间
 */
function requestHostTimeout(callback, ms) {
  taskTimeoutID = localSetTimeout(() => {
    callback(getCurrentTime());
  }, ms);
  
  /**
   * 因此，上面的代码，就可以看作是
   * id = setTimeout(function(){
   *    handleTimeout(getCurrentTime())
   * }, ms)
   */
}
```

可以看到 `requestHostTimeout` 本质上就是调用 `setTimeout`，然后在 `setTimeout` 中调用传入的 `handleTimeout` 。

## handleTimeout

```js
/**
 *
 * @param {*} currentTime 当前时间
 */
function handleTimeout(currentTime) {
  isHostTimeoutScheduled = false;
  
  // 遍历timerQueue，将时间已经到了的延时任务放入到 taskQueue
  advanceTimers(currentTime);

  if (!isHostCallbackScheduled) {
    if (peek(taskQueue) !== null) {
      // 从普通任务队列中拿一个任务出来
      isHostCallbackScheduled = true;
      // 采用调度普通任务的方式进行调度
      requestHostCallback(flushWork);
    } else {
      // taskQueue任务队列里面是空的
      // 再从 timerQueue 队列取一个任务出来
      // peek 是小顶堆中提供的方法
      const firstTimer = peek(timerQueue);
      if (firstTimer !== null) {
        // 取出来了，接下来取出的延时任务仍然使用 requestHostTimeout 进行调度
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
      }
    }
  }
}
```

这个函数的关键点：
- `handleTimeout()` 依旧使用的是 `advanceTimers()` 方法，该方法的作用就是将已经到时间的延时任务放入到 `taskQueue` 中；
- 然后使用 `requestHostCallback()` 进行任务调度；
- 如果从 `taskQueue` 取出的任务为空，则再次从 `timerQueue` 中取出一个延时任务，最后使用 `requestHostTimeout()` 进行任务调度；

## 流程图

整个 Scheduler 任务调度流程图如下：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-30-023505.png)