---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/19 React 渲染流程/","dg-note-properties":{}}
---

现代前端框架都可以总结为一个公式： UI = f（state）。
上面的公式可以进行一个拆分：
- 根据自变量 State 的变化计算出 UI 的变化；
- 根据 UI 的变化执行具体的宿主环境 API；

对应的公式：

```js
const state = reconcile(update); // 通过 reconciler 计算出最新的状态
const UI = commit(state); // 根据上一步计算出来的 state 渲染出 UI
```

对应到 React 里面就两大「阶段」：
- Render 阶段：调合虚拟 DOM，计算出最终要渲染的虚拟 DOM；
- Commit 阶段：根据上一步计算出来的虚拟 DOM 渲染具体的 UI；

每个阶段对应不同的组件：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-23-101849.png)
- 调度器 Scheduler：调度任务，为任务进行优先级排序，让优先级高的任务进入到 Reconciler；
- 协调器 Reconciler：生成 Fiber 对象，收集副作用，找出哪些节点发生了变化，打上不同的 Flags，diff 算法也是在这个组件中执行的；
- 渲染器 Renderer：根据协调器计算出的虚拟 DOM 同步渲染节点到视图上；

看一个示例：

```js
export default () => {
  const [count, updateCount] = useState(0);
  return (
    <ul>
	  <button onClick={() => updateCount(count + 1)}>乘以{count}</button>
      <li>{1 * count}</li>
      <li>{2 * count}</li>
      <li>{3 * count}</li>
    </ul>
  );
}
```

当用户点击了按钮，首先是由 Scheduler 进行任务的协调，render 阶段（虚线框内）的工作流程是可以随时被中断的，中断原因：
- 有其他更高优先级的任务需要执行；
- 当前的 time slice 没有剩余时间；
- 发生了其他的错误；

需要注意上面的 render 阶段的工作是在内存中进行的，不会更新到宿主环境 UI，所以这个阶段即使工作流程被反复中断，用户也不会看到“更新不完整的 UI”。

当 Scheduler 调度完成后，将任务交给 Reconciler，Reconciler 就需要计算出新的 UI，最后由 Renderer 「同步」进行渲染并更新操作。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-23-103449.png)
## 调度器

在 React16 版本之前，采用的是 Stack 结构，所有的任务只能同步进行，无法被中断，这就导致浏览器可能出现掉帧的现象。React 为了解决这个问题，从 16 版本开始进行两大更新：
- 引入 Fiber
- 新增 Scheduler 调度器

Scheduler 在浏览器的原生 API 中实际上是有类似实现的，这个 API 就是 `requestIdleCallback`。

```cardlink
url: https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback
title: "requestIdleCallback - Web API | MDN"
description: "window.requestIdleCallback() 方法插入一个函数，这个函数将在浏览器空闲时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间timeout，则有可能为了在超时前执行函数而打乱执行顺序。"
host: developer.mozilla.org
favicon: https://developer.mozilla.org/favicon.ico
```

```js
// 模拟延迟
function delay(duration) {
  const start = Date.now();
  while (Date.now() - start < duration) {}
}

// 存放任务的数组
const taskList = [];

// 添加任务
for (let i = 0; i < 10; i++) {
  taskList.push(() => {
    delay(10);
    console.log(`执行任务${i}`);
  });
}

// 执行任务
function executeTask(deadline) {
  console.log(`当前帧渲染完成后的剩余时间为${deadline.timeRemaining()}`);
  
  // 每一帧渲染完成后的剩余时间，如果时间充足就执行任务
  while (deadline.timeRemaining() > 0 && taskList.length > 0) {
    taskList.shift()();
  }
  
  // 没有进入 while 循环，说明当前帧渲染完成后的剩余时间不足，但是任务列表依旧存在任务
  // 需要等下一帧渲染后再接着执行
  if (taskList.length > 0) {
    window.requestIdleCallback(executeTask);
  }
}

// 首次启动，浏览器绘制后有空闲时间开始执行 executeTask
window.requestIdleCallback(executeTask);
```
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260907143932.png)
上面的示例中，页面有一个来回移动的小球，同时我们在 JS 中使用 `requestIdleCallback()` 来执行 JS 的任务，从打印的结果中可以看出，每次浏览器绘制完成后都会执行 N 次任务。

虽然浏览器有类似的 API，但是因为这个 API 存在兼容问题 React 团队也就没有使用这个 API，因此团队内部实现了这样的一套机制，这个就是调取器 Scheduler。

```cardlink
url: https://github.com/react/react/blob/main/packages/scheduler/README.md
title: "react/packages/scheduler/README.md at main · react/react"
description: "The library for web and native user interfaces. Contribute to react/react development by creating an account on GitHub."
host: github.com
favicon: https://github.githubassets.com/favicons/favicon.svg
image: https://opengraph.githubassets.com/84f52c9ad54b17edc00654f48f20fcc0e945702b32c8d8fcac741ddc4e92a244/react/react
```

## 协调器

协调器是 render 阶段的第二阶段工作，类组件或函数组件本身就在这个阶段被调用的。
根据 Scheduler 的调度结果不同，协调器的起点可能也不同（简单理解就是 Scheduler 决定这次更新是应该一口气干完，还是允许干一会歇一会）：
- `performSyncWorkOnRoot` 同步更新流程
- `performConcurrentWorkOnRoot` 并发更新流程

```js
// performSyncWorkOnRoot 会执行该方法
function workLoopSync(){
  while(workInProgress !== null){
    // 执行工作单元
    performUnitOfWork(workInProgress)
  }
}

// performConcurrentWorkOnRoot 会执行该方法
function workLoopConcurrent(){
  while(workInProgress !== null && !shouldYield()){
	// 执行工作单元
    performUnitOfWork(workInProgress)
  }
}
```

新的架构使用 Fiber 对象来描述 DOM 结构，最终需要形成一颗 FiberTree，只不过这颗树使用的是链表的形式串联在一起。

`workInProgress` 代表的是当前的 FiberNode。

`performUnitOfWork()` 执行工作单元函数会创建下一个 FiberNode，并且还会将已创建的 FiberNode 连接起来（child、return、sibling），从而形成一个链表结构的 FiberTree。

如果 `workInProgress` 为空，说明已经没有下一个 FiberNode 了，也就说明整棵 FiberTree 已经构建完成了。

上面代码中，唯一的差异就是是否调用了 `shouldYield()` 方法，这个方法表明是否可以中断。

`performUnitOfWork()` 执行工作单元函数在创建下一个 FiberNode 的时候，整体上的工作流程可以分为两大块：
- 递阶段
- 归阶段

### 递阶段

递阶段会从 HostRootFiber 开始向下以深度优先为原则进行遍历，遍历到的每一个 FiberNode 执行 `beginWork()` 方法。该方法会根据传入的 FiberNode 创建下一级 FiberNode，此时可能存在两种情况：

1、下一级只有一个元素，`beginWork()` 方法会创建对应的 FiberNode，并于 workInProgress 连接。

```html
<ul>
  <li></li>
</ul>
```

这里就会创建 li 对应的 FiberNode，做出如下的连接：

```js
LiFiber.return = UlFiber;
```

2、下一级存在多个元素，这时 `beginWork()` 方法会依次创建所有的子 FiberNode 并且通过 sibling 连接到一起，每个子 FiberNode 也会和 workInProgress 连接。

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

此时会创建 3 个 li 对应的 FiberNode，连接情况如下：

```js
// 所有的子 Fiber 依次连接
Li0Fiber.sibling = Li1Fiber;
Li1Fiber.sibling = Li2Fiber;

// 子 Fiber 还需要和父 Fiber 连接
Li0Fiber.return = UlFiber;
Li1Fiber.return = UlFiber;
Li2Fiber.return = UlFiber;
```

由于采用的是深度优先的原则，当无法再往下走的时候，会进入到归阶段。

### 归阶段

归阶段会调用 `completeWork()` 方法来处理 FiberNode，做一些副作用的收集。

当某个 FiberNode 执行完成了 `completeWork()` 方法后，如果存在兄弟元素，就会进入到兄弟元素的递阶段，如果不存在兄弟元素，就会进入父 FiberNode 的归阶段。

```js
function performUnitOfWork(fiberNode){
  // 省略 beginWork 
  if(fiberNode.child){
    performUnitOfWork(fiberNode.child);
  }
  
  // 省略 CompleteWork 
  if(fiberNode.sibling){
    performUnitOfWork(fiberNode.sibling);
  }
}
```

使用一张图示意：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-24-031518.png)
以下是 AI 总结的流程图：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/ChatGPT_Image_2026%E5%B9%B49%E6%9C%887%E6%97%A5_15_39_28.png)

## 渲染器

Renderer 工作的阶段称之为 commit 阶段，这个阶段会将各种副作用 commit 到宿主环境的 UI 中。

相较于之前的 render 阶段可以被打断，commit 阶段一旦开始就会同步执行，直到完成渲染工作。

整个渲染器渲染过程可以分为三个子阶段：
- BeforeMutation 阶段；
- Mutation 阶段；
- Layout 阶段；
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-02-090354.png)