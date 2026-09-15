---
{"dg-publish":true,"permalink":"//web/4/react18/22-scheduler/","dg-note-properties":{}}
---

源码地址：

```cardlink
url: https://github.com/react/react/blob/main/packages/scheduler/src/forks/Scheduler.js
title: "react/packages/scheduler/src/forks/Scheduler.js at main · react/react"
description: "The library for web and native user interfaces. Contribute to react/react development by creating an account on GitHub."
host: github.com
favicon: https://github.githubassets.com/favicons/favicon.svg
image: https://opengraph.githubassets.com/b8551d54ca2e0a2f9ea79d70d8c841e408fd146ba2d6e14a83693e173a8bee34/react/react
```

## unstable_scheduleCallback

该函数的主要目的就是调度任务，函数的分析如下：

```js hl:67-74,104-119
let getCurrentTime = () => performance.now();

// 有两个队列分别存储普通任务和延时任务
// 里面采用了一种叫做小顶堆的算法，保证每次从队列里面取出来的都是优先级最高（时间即将过期）
var taskQueue = []; // 存放普通任务
var timerQueue = []; // 存放延时任务

var maxSigned31BitInt = 1073741823;

// Timeout 对应的值
var IMMEDIATE_PRIORITY_TIMEOUT = -1;
var USER_BLOCKING_PRIORITY_TIMEOUT = 250;
var NORMAL_PRIORITY_TIMEOUT = 5000;
var LOW_PRIORITY_TIMEOUT = 10000;
var IDLE_PRIORITY_TIMEOUT = maxSigned31BitInt;

/**
 *
 * @param {*} priorityLevel 优先级等级
 * @param {*} callback 具体要做的任务
 * @param {*} options { delay: number } 这是一个对象，该对象有 delay 属性，表示要延迟的时间
 * @returns
 */
function unstable_scheduleCallback(priorityLevel, callback, options) {
  // 获取当前的时间
  var currentTime = getCurrentTime();

  var startTime;
  // 整个这个 if.. else 就是在设置起始时间，如果有延时，起始时间需要添加上这个延时
  if (typeof options === "object" && options !== null) {
    var delay = options.delay;
    // 如果设置了延时时间，那么 startTime 就为当前时间 + 延时时间
    if (typeof delay === "number" && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }

  var timeout;
  // 根据传入的优先级等级来设置不同的 timeout
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }
  // 接下来就计算出过期时间
  // 计算出来的时间有些比当前时间要早，绝大部分比当前的时间要晚一些
  var expirationTime = startTime + timeout;

  // 创建一个新的任务
  var newTask = {
    id: taskIdCounter++, // 任务 id
    callback, // 该任务具体要做的事情
    priorityLevel, // 任务的优先级别
    startTime, // 任务开始时间
    expirationTime, // 任务的过期时间
    sortIndex: -1, // 用于后面在小顶堆（这是一种算法，可以始终从任务队列中拿出最优先的任务）进行排序的索引
  };
  
  if (enableProfiling) {
    newTask.isQueued = false;
  }

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

`unstable_scheduleCallback()` 函数的关键点：
- 函数内维护了两个任务队列，一个是 `taskQueye` 存放已经可以执行的任务，一个是 `timerQueye` 存放未来要执行的任务；
- 任务队列使用 `sortIndex` 来进行小顶堆的排序（按照任务的时间进行从近到远进行排序），回头确保在 `peek()` 取出的任务始终是时间优先级最高的那个任务；
- 根据传入的 `priorityLevel` 优先级属性，会进行不同的 `timeout` 的设置，任务的 `timeout` 时间也就不一样了，有的比当前时间要小，也就表示这个任务需要立即执行，绝大部分的时间比当前时间大；
	- ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-29-065931.png)
- 不同的任务最终调用的函数不一样：
	- 普通任务调用 `requestHostCallback()` 进行任务的调度；
	- 延时任务调用 `requestHostTimeout()` 使任务到达时间点时把延时任务推入 `taskQueue` 中；

## requestHostCallback 和 schedulePerformWorkUntilDeadline

```js hl:14-15,28-36
/**
 * 
 * @param {*} callback 是在调用的时候传入的 flushWork
 * requestHostCallback 这个函数没有做什么事情，主要就是调用 schedulePerformWorkUntilDeadline
 */
function requestHostCallback(callback) {
  // 仅保存要执行的任务，但不执行
  // scheduledHostCallback ---> flushWork
  scheduledHostCallback = callback;
  
  // 判断调度是否正在运行
  if (!isMessageLoopRunning) {
    isMessageLoopRunning = true;
    // 实例化 MessageChannel 进行后面的调度
    schedulePerformWorkUntilDeadline(); 
  }
}

// 下面的代码就是根据不同的宿主环境，赋予不同的函数
let schedulePerformWorkUntilDeadline; // undefined
if (typeof localSetImmediate === 'function') {
  // Node.js and old IE.
  // https://github.com/facebook/react/issues/20756
  schedulePerformWorkUntilDeadline = () => {
    localSetImmediate(performWorkUntilDeadline);
  };
} else if (typeof MessageChannel !== 'undefined') {
  // 浏览器环境
  // 大多数情况下，使用的是 MessageChannel
  const channel = new MessageChannel();
  const port = channel.port2;
  channel.port1.onmessage = performWorkUntilDeadline;
  schedulePerformWorkUntilDeadline = () => {
	// 调用 schedulePerformWorkUntilDeadline，这里会创建一个宏任务
    port.postMessage(null);
  };
} else {
  // 低版本浏览器环境
  // setTimeout 进行兜底
  schedulePerformWorkUntilDeadline = () => {
    localSetTimeout(performWorkUntilDeadline, 0);
  };
}
```

这段代码的关键点：
- `requestHostCallback()` 函数主要就是调用了 `schedulePerformWorkUntilDeadline()` 函数；
- `schedulePerformWorkUntilDeadline` 最开始是一个遍历，后面通过不同的环境赋予不同的函数；
- `performWorkUntilDeadline` 才是真正进入 Scheduler 工作循环；

## performWorkUntilDeadline

```js hl:25,27-32
let startTime = -1;

const performWorkUntilDeadline = () => {
  // scheduledHostCallback ---> flushWork
  if (scheduledHostCallback !== null) {
    // 获取当前的时间
    const currentTime = getCurrentTime();
    // 这里的 startTime 并非 unstable_scheduleCallback 方法里面的 startTime
    // 而是一个全局变量，默认值为 -1
    // 主要用来测量任务的执行时间，从而能够知道主线程被阻塞了多久
    startTime = currentTime;
    const hasTimeRemaining = true; // 默认还有剩余时间

    // If a scheduler task throws, exit the current browser task so the
    // error can be observed.
    //
    // Intentionally not using a try-catch, since that makes some debugging
    // techniques harder. Instead, if `scheduledHostCallback` errors, then
    // `hasMoreWork` will remain true, and we'll continue the work loop.
    let hasMoreWork = true; // 默认还有需要做的任务
    try {
      // scheduledHostCallback ---> flushWork(true, 开始时间): boolean
      // 如果是 true，代表工作没做完
      // false 代表没有任务了
      hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);
    } finally {
      if (hasMoreWork) {
        // If there's more work, schedule the next message event at the end
        // of the preceding one.
        // 如果有更多的任务
        // 那么就使用 messageChannel 进行一个 message 事件的调度，就将任务放入到任务队列里面
        schedulePerformWorkUntilDeadline();
      } else {
        // 说明任务做完了
        isMessageLoopRunning = false;
        scheduledHostCallback = null; // scheduledHostCallback 之前为 flushWork，设置为 null
      }
    }
  } else {
    isMessageLoopRunning = false;
  }
  // Yielding to the browser will give it a chance to paint, so we can
  // reset this.
  needsPaint = false;
};
```

`performWorkUntilDeadline()` 函数的关键点：
- 这个方法主要就是调用了前面传入的 `flushWork` 方法（`scheduledHostCallback = flushWork`），调用后会返回一个布尔值，根据这个布尔值来判断是否还有剩余的任务。如果有再次调用 `schedulePerformWorkUntilDeadline()` 进行宏任务的包装；

## flushWork 和 workLoop

`flushWork()` 才是真正的任务消费逻辑：

```js hl:72-89,101-102,110-111,115-126
/**
 *
 * @param {*} hasTimeRemaining 是否有剩余的时间，一开始是 true
 * @param {*} initialTime 做这一个任务时开始执行的时间
 * @returns
 */
function flushWork(hasTimeRemaining, initialTime) {
  // ...
  try {
    if (enableProfiling) {
      try {
        // 核心实际上是这一句，调用 workLoop
        return workLoop(hasTimeRemaining, initialTime);
      } catch (error) {
        // ...
      }
    } else {
      // 核心实际上是这一句，调用 workLoop
      return workLoop(hasTimeRemaining, initialTime);
    }
  } finally {
    // ...
  }
}

/**
 *
 * @param {*} hasTimeRemaining 是否有剩余的时间，一开始是 true
 * @param {*} initialTime 做这一个任务时开始执行的时间
 * @returns
 */
function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  // 该方法实际上是用来遍历 timerQueue，判断是否有已经到期了的任务
  // 如果有，将这个任务放入到 taskQueue
  advanceTimers(currentTime);
  
  // 从 taskQueue 里面取一个任务出来
  currentTask = peek(taskQueue);
  
  // 如果取出的任务不为空，并且 Scheduler 没有暂停
  while (
    currentTask !== null &&
    !(enableSchedulerDebugging && isSchedulerPaused)
  ) {
  
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // This currentTask hasn't expired, and we've reached the deadline.
      // currentTask.expirationTime > currentTime 表示任务还没有过期
      // hasTimeRemaining 代表是否有剩余时间
      // shouldYieldToHost 任务是否应该暂停，归还主线程
      // 那么我们就跳出 while
      break;
    }
    
    // 没有进入到上面的 if，说明这个任务到过期时间
    // 并且有剩余时间来执行，没有到达需要浏览器渲染的时候
    // 那我们就执行该任务即可
    const callback = currentTask.callback; // 拿到这个任务
    if (typeof callback === "function") {
      // 说明当前的任务是一个函数，我们执行该任务

      currentTask.callback = null;
      currentPriorityLevel = currentTask.priorityLevel;
      const didUserCallbackTimeout = currentTask.expirationTime <= currentTime; // 任务是否过期
      if (enableProfiling) {
        markTaskRun(currentTask, currentTime);
      }
      
      // 任务的执行实际上就是在这一句
      const continuationCallback = callback(didUserCallbackTimeout);
      currentTime = getCurrentTime();
      
      // 如果返回一个函数，说明任务没有完成，后面会继续处理
      if (typeof continuationCallback === "function") {
        // If a continuation is returned, immediately yield to the main thread
        // regardless of how much time is left in the current time slice.
        // $FlowFixMe[incompatible-use] found when upgrading Flow
        // 把“剩下的工作”保存回当前任务中
        currentTask.callback = continuationCallback;
        if (enableProfiling) {
          // $FlowFixMe[incompatible-call] found when upgrading Flow
          markTaskYield(currentTask, currentTime);
        }
        advanceTimers(currentTime);
        // 直接跳出 workLoop
        return true;
      } else {
	    // 如果没有返回函数，说明当前任务处理完成了
	    
        if (enableProfiling) {
          // $FlowFixMe[incompatible-call] found when upgrading Flow
          markTaskCompleted(currentTask, currentTime);
          // $FlowFixMe[incompatible-use] found when upgrading Flow
          currentTask.isQueued = false;
        }
        
        if (currentTask === peek(taskQueue)) {
	      // 把任务移除掉
          pop(taskQueue);
        }
        advanceTimers(currentTime);
      }
    } else {
      // 直接弹出
      pop(taskQueue);
    }
    // 再从 taskQueue 里面拿一个任务出来
    currentTask = peek(taskQueue);
  }
  
  // Return whether there's additional work
  if (currentTask !== null) {
    // 如果不为空，代表还有更多的任务，那么回头外部的 hasMoreWork 拿到的就也是 true
    return true;
  } else {
    // taskQueue 这个队列是空了，那么我们就从 timerQueue 里面去看延时任务
    const firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }
    // 没有进入上面的 if，说明 timerQueue 里面的任务也完了，返回 false，回头外部的 hasMoreWork 拿到的就也是 false
    return false;
  }
}
```

整个 while 循环的过程可以理解为：

```md
拿任务
  ↓
是否应该让出线程？
  ↓
 否
  ↓
执行任务
  ↓
任务完成了吗？
  ↓
完成 → 删除
没完成 → 保存 continuation
  ↓
检查新的延时任务
  ↓
再拿下一个任务
```

上面代码片段的关键点：
- `flushWork()` 主要就是调用 `workLoop()` ;
- `workLoop()` 有一个循环，这个循环保证能够从任务队列中不停的取出任务；
- 每次取出一个新任务的时候，都要经过判断：

```js
if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
 ) 

```

- `currentTask.expirationTime > currentTime` 表示任务还没有过期；
- `hasTimeRemaining` 表示还有剩余的时间；
- `shouldYieldToHost()` 表示是否应该暂停执行，让出主线程；
- 如果进入了 if 条件，说明某些原因不能继续执行任务，需要理解归还主线程，那么就跳出 while 循环；

## shouldYieldToHost

这个函数的主要作用就是：判断任务应该中断，让出主线程。

```js
function shouldYieldToHost() {
	// getCurrentTime 获取当前时间
    // startTime 是我们任务开始时的时间，一开始是 -1，之后任务开始时，将任务开始时的时间复值给了它
  const timeElapsed = getCurrentTime() - startTime;
  // frameInterval 默认设置的是 5ms
  if (timeElapsed < frameInterval) {
    // 如果小于默认设置的 5ms
    // 说明主线程只被阻塞了一点点时间，远远没达到需要归还的时候
    return false;
  }
  
  // 如果没有进入上面的 if，说明主线程已经被阻塞了一段时间了
  // 需要归还主线程
  if (enableIsInputPending) {
    if (needsPaint) {
      // There's a pending paint (signaled by `requestPaint`). Yield now.
      return true;
    }
    
    if (timeElapsed < continuousInputInterval) {
      // We haven't blocked the thread for that long. Only yield if there's a
      // pending discrete input (e.g. click). It's OK if there's pending
      // continuous input (e.g. mouseover).
      if (isInputPending !== null) {
        return isInputPending();
      }
    } else if (timeElapsed < maxInterval) {
      // Yield if there's either a pending discrete or continuous input.
      if (isInputPending !== null) {
        return isInputPending(continuousOptions);
      }
    } else {
      // We've blocked the thread for a long time. Even if there's no pending
      // input, there may be some other scheduled work that we don't know about,
      // like a network event. Yield now.
      return true;
    }
  }

  // `isInputPending` isn't available. Yield now.
  return true;
}
```

这段代码的关键点：
- 首先计算出 `timeElapsed` 然后判断是否超时，如果没有超时就返回 `false`，表示不需要让出主线程，否则返回 `true`，表示需要让出主线程；
- `frameInterval` 默认设置的是 5ms;

## advanceTimers

`advanceTimers` 函数是延时任务转移到普通任务队列的核心函数：

```js
function advanceTimers(currentTime) {
  // Check for tasks that are no longer delayed and add them to the queue.
  // 从 timerQueue 队列里面获取一个任务
  let timer = peek(timerQueue);
  // 遍历整个 timerQueue
  while (timer !== null) {
    if (timer.callback === null) {
      // 这个任务没有对应的要执行的 callback，直接从这个队列弹出
      pop(timerQueue);
    } else if (timer.startTime <= currentTime) {
      // 进入这个分支，说明当前的任务已经不再是延时任务，需要立即执行
      // 我们需要将其转移到 taskQueue
      pop(timerQueue);
      timer.sortIndex = timer.expirationTime;
      push(taskQueue, timer); // 推入到 taskQueue
      // ...
    } else {
      return;
    }
    // 从 timerQueue 里面再取一个新的进行判断
    timer = peek(timerQueue);
  }
}
```

`advanceTimers()` 方法就是遍历 `timerQueue` 延时任务队列，查看是否存在已经过期的任务，如果存在的话，那么就将这个任务添加到 `taskQueue` 队列中，等待执行。

## 总结

| 函数                                 | 职责                |
| ---------------------------------- | ----------------- |
| `unstable_scheduleCallback`        | 创建 Scheduler 任务   |
| `timerQueue`                       | 存放未来执行任务          |
| `taskQueue`                        | 存放马上执行任务          |
| `advanceTimers`                    | 把到期任务搬入 taskQueue |
| `requestHostCallback`              | 注册 Scheduler 工作入口 |
| `schedulePerformWorkUntilDeadline` | 安排浏览器未来执行         |
| `performWorkUntilDeadline`         | 真正进入 Scheduler    |
| `flushWork`                        | 执行任务调度            |
| `workLoop`                         | 循环执行任务            |
| `shouldYieldToHost`                | 判断是否暂停让浏览器工作      |