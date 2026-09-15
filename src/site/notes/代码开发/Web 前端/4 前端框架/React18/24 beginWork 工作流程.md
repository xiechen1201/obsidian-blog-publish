---
{"dg-publish":true,"permalink":"//web/4/react18/24-begin-work/","dg-note-properties":{}}
---

`beginWork()` 属于 Reconciler 协调器，是 Render 阶段的第二阶段工作，整个工作过程可以分为“递”和“归”：
- 递：`beginWork()`
- 归：`completeWork()`
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-10-053722.png)
`beginWork()` 方法主要是根据传入的 FiberNode 创建下一级的 FiberNode。

整个 `beginWork()` 方法流程如下：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-01-015305.png)
首先在 `beginWork()` 中，会判断当前的流程是 mount（初次渲染）还是 update（更新），判断的依据就是 current FiberNode 是否存在。

```js
if(current !== null){
  // 说明 CurrentFiberNode 存在，应该是 update
} else {
  // 应该是 mount
}
```

如果是 update 接下来就会判断 workInProgress FiberNode 能否进行复用，如果不可以，那么 update 和 mount 的流程基本上一致：
- 根据 wip.tag 进行不同分支的处理；
- 根据 reconcile 算法生成下一级的 FiberNode（diff 算法）

无法复用的 update 流程和 mount 流程基本一致，主要区别在于是否会生成带副作用标记 flags 的 FiberNode。

`beginWork()` 方法如下：

```js
// current 代表的是 current FiberNode
// workInProgress 代表的是 workInProgress FiberNode，后面会简称为 wip FiberNode
function beginWork(current, workInProgress, renderLanes) {
  // ...
  if(current !== null) {
    // 进入此分支，说明是更新
  } else {
    // 说明是首次渲染
  }
  
  // ...
  
  // 根据不同的 tag，进入不同的处理逻辑
  switch (workInProgress.tag) {
    case IndeterminateComponent: {
      // ...
    }
    case FunctionComponent : {
      // ...
    }
    case ClassComponent : {
      // ...
    }
  }
}
```

> [!info]
> 关于 tag，在 React 源码中定义了 28 种 tag：
> ```js
>export const FunctionComponent = 0;
>export const ClassComponent = 1;
>export const IndeterminateComponent = 2; // Before we know whether it is function or class
>export const HostRoot = 3; // Root of a host tree. Could be nested inside another node.
>export const HostPortal = 4; // A subtree. Could be an entry point to a different renderer.
>export const HostComponent = 5;
>export const HostText = 6;
>export const Fragment = 7;
>// ...
>```
>
>不同的 FiberNode 对应不同的 tag：
>- HostComponent 代表的就是原生组件（div、span、p）
>- FunctionComponent 代表的是函数组件
>- ClassComponent 代表的是类组件

根据不同的 tag 处理完 FiberNode 之后，根据是 mount 还是 update 会进入到不同的方法：
- mount：`mountChildFibers`
- update：`reconcileChildFibers`

这两个方法实际上都是一个叫做 `ChildReconciler()` 方法的返回值：

```js
var reconcileChildFibers = ChildReconciler(true);
var mountChildFibers = ChildReconciler(false);

function ChildReconciler(shouldTrackSideEffects) {}
```

也就是说，在 `ChildReconciler()` 方法上，`shouldTrackSideEffects` 是一个布尔值：
- false：不追踪副作用，不做 flags 标记，因为是 mount 阶段；
- true：要追踪副作用，做 flags 标记，因为是 update 阶段；

> [!tip] 
> 如何理解这里的副作用？
> Fiber 在 Render 阶段计算过程中，发现这个节点未来在 Commit 阶段需要对真实 DOM 做某些操作，于是提前打一个标记。
> 
> 也就说 Render 阶段负责发现变化，Commit 阶段负责执行变化。

在 `ChildReconciler()` 方法内部，就会根据 `shouldTrackSideEffects` 做一些不同的处理：

```js
function placeChild(newFiber, lastPlacedIndex, newIndex){
  newFiber.index = newIndex;
  
  if(!shouldTrackSideEffects){
    // 说明是初始化
    // 说明不需要标记 Placement
    newFiber.flags |= Forked; // 位运算
    return lastPlacedIndex
  }
  // ...
  // 说明是更新
  // 标记为 Placement
  newFiber.flags |= Placement; // 位运算
  
}
```

可以看到`beginWork()` 方法内部，也会做一些 flags 标记（主要是在 update 阶段），这些 flags 标记主要和元素的位置有关系：
- 标记 `ChildDeletion` 表示删除操作；
- 标记 `Placement` 表示插入或移动操作；