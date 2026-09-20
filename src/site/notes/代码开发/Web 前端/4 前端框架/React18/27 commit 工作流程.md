---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/27 commit 工作流程/","dg-note-properties":{}}
---

整个 React 的工作流程可以分为两个阶段：
- Render 阶段
	- Scheduler
	- Reconciler
- Commit 阶段

一句话总结就是：Render 负责“算”，Commit 负责“干”。

注意，Render 阶段是在内存中运行的，这意味着可以被打断，而 commit 阶段则一旦开始就会「同步」执行完成。

Commit 阶段又可以分为 3 个子阶段：
- BeforeMutation 阶段（DOM 修改之前）
- Mutation 阶段（DOM 修改）
- Layout 阶段（DOM 修改完成）

整体流程图如下：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-02-090326.png)
每个阶段再次分为 3 个子阶段：
- commitXXXEffects
- commitXXXEffects_begin
- commitXXXEffects_complete

之所以要分出 3 个子阶段，是因为有一些共同的事情需要完成。

1、commitXXXEffects()

这个函数是每个子阶段的入口函数，`finishedWork` 会作为 `firstChild` 参数传入进去，代码如下：

```js
function commitXXXEffects(root, firstChild){
  nextEffect = firstChild;
  // 省略标记全局变量
  commitXXXEffects_begin();
  // 省略重置全局变量
}
```

因此这个函数的主要工作就是将 `firstChild` 赋值给全局变量 `nextEffect`，然后执行 `commitXXXEffects_begin()` 方法。

2、commitXXXEffects_begin()

这个函数==主要是向下遍历 FiberNode==，遍历时会一直向下，直到第一个满足如下条件之一的 FiberNode：
- 当前的 FiberNode 的子 FiberNode 不包含该子阶段对应的 flags；
- 当前的 FiberNode 不存在子 FiberNode；

接下来会对目标的 FiberNode 执行 `commitXXXEffects_complete()` 方法，`commitXXXEffects_begin()` 代码如下：

```js
function commitXXXEffects_begin(){
  while(nextEffect !== null) {
    let fiber = nextEffect;
    let child = fiber.child;
    
    // 省略该子阶段的一些特有操作
    
    if(fiber.subtreeFlags !== NoFlags && child !== null){
      // 继续向下遍历
      nextEffect = child;
    } else {
      commitXXXEffects_complete();
    }
  }
}
```

3、commitXXXEffects_complete()

该方法==主要是针对 flags 做具体的操作了==，主要包含三个步骤：
- 对当前的 FiberNode 执行 flags 对应的操作，也就是执行 `commitXXXEffectsOnFiber()` ；
- 如果当前的 FiberNode 存在兄弟 FiberNode，则对兄弟 FiberNode 执行 `commitXXXEffects_begin()` ；
- 如果不存在兄弟 FiberNode，则对父 FiberNode 执行 `commitXXXEffects_complete()` ；

相关代码如下：

```js
function commitXXXEffects_complete(root){
  while(nextEffect !== null){
    let fiber = nextEffect;
    
    try{
      commitXXXEffectsOnFiber(fiber, root);
    } catch(error){
      // 错误处理
    }
    
    let sibling = fiber.sibling;
    
    if(sibling !== null){
      // ...
      nextEffect = sibling;
      return
    }
    
    nextEffect = fiber.return;
  }
}
```

总结一下，每个子阶段都会以 DFS 的原则进行遍历，最终会在 `commitXXXEffectsOnFiber()` 中针对不同的 flags 做出不同的处理；

> 整个流程和 Reconciler 中的 `beginWork()` 和 `completeWork()` 非常的相似。
> 都是深度优先，处理兄弟节点，然后往上返回。

## BeforeMutation 阶段

BeforeMutation 阶段的主要工作发生在 `commitBeforeMutationEffects_complete()` 中的 `commitBeforeMutationEffectsOnFiber()` 方法，代码如下：

```js hl:16,28
function commitBeforeMutationEffectsOnFiber(finishedWork){
  const current = finishedWork.alternate;
  const flags = finishedWork.flags;
  
  //...
  // Snapshot 表示 ClassComponent 存在更新，且定义了 getSnapshotBeforeUpdate 方法
  if(flags & Snapshot !== NoFlags) {
    switch(finishedWork.tag){
      case ClassComponent: {
        if(current !== null){
          const prevProps = current.memoizedProps;
          const prevState = current.memoizedState;
          const instance = finishedWork.stateNode;
          
          // 执行 getSnapshotBeforeUpdate
          const snapshot = instance.getSnapshotBeforeUpdate(
          	finishedWork.elementType === finishedWork.type ? 
            prevProps : resolveDefaultProps(finishedWork.type, prevProps),
            prevState
          )
        }
        break;
      }
      case HostRoot: {
        // 清空 HostRoot 挂载的内容，方便 Mutation 阶段渲染
        if(supportsMutation){
          const root = finishedWork.stateNode;
          clearContainer(root.containerInfo);
        }
        break;
      }
    }
  }
}
```

上面的代码流程中，主要处理下面两种类型的 FiberNode：

- ClassComponent：执行 `getSnapshotBeforeUpdate()` 方法；
- HostRoot：清空 HostRoot 挂载的内容，方便 Mutation 阶段进行渲染；

## Mutation 阶段

对于 HostComponent，Mutation 阶段的主要工作就是对 DOM 元素进行增、删、改。

1、删除 DOM 元素

```js hl:5,11
function commitMutationEffects_begin(root){
  while(nextEffect !== null){
    const fiber = nextEffect;
    // 删除 DOM 元素
    const deletions = fiber.deletions;
    
    if(deletions !== null){
      for(let i=0;i<deletions.length;i++){
        const childToDelete = deletions[i];
        try{
          commitDeletion(root, childToDelete, fiber);
        } catch(error){
          // 省略错误处理
        }
      }
    }
    
    const child = fiber.child;
    if((fiber.subtreeFlags & MutationMask) !== NoFlags && child !== null){
      nextEffect = child;
    } else {
      commitMutationEffects_complete(root);
    }
  }
}
```

删除 DOM 元素的操作发生在 `commitMutationEffects_begin()` 方法中，首先会拿到 `deletions` 数组，之后遍历这个数组进行删除操作，对应删除 DOM 元素的方法为 `commitDeletion()`。

`commitDeletion()` 方法内部的完整逻辑实际上是比较复杂的，原因是因为在删除一个 DOM 元素时，不是说删除就直接删除了，还需要考虑下面的一些因素：
- 其子树中所有组件的 mount 逻辑；
- 其子树中所有的 ref 属性卸载操作；
- 其子树中所有的 effect 相关 Hook 的 destroy 回调执行；

例如有下面的代码：

```jsx
<div>
  <SomeClassComponent/>
  <div ref={divRef}>
  	<SomeFunctionComponent/>
  </div>
</div>
```

当删除最外层 div 元素的时候，需要考虑：
- 执行 `<SomeClassComponent />` 类组件对应的 `componentWillUnmount()` 方法；
- 执行 `<SomeFunctionComponent />` 函数组件中的 `useEffect`、`useLayoutEffect` 这些 Hook 中的 destroy 方法；
- divRef 的卸载操作；

整个删除操作都是以 DFS 的顺序，遍历子树的每个 FiberNode，执行对应的操作。

2、插入、移动 DOM 元素

上面的删除操作是在 `commitMutationEffects_begin()` 方法内执行的，而插入、移动 DOM 元素则是在 `commitMutationEffects_complete()` 方法内的 `commitMutationEffectsOnFiber()` 方法内执行的，相关代码：

```js hl:11,18
function commitMutationEffectsOnFiber(finishedWork, root){
  const flags = finishedWork.flags;

  // ...
  
  const primaryFlags = flags & (Placement | Update | Hydrating);
  
  outer: switch(primaryFlags){
    case Placement:{
      // 执行 Placement 对应操作
      commitPlacement(finishedWork);
      // 执行完 Placement 对应操作后，移除 Placement flag
      finishedWork.flags &= ~Placement;
      break;
    }
    case PlacementAndUpdate:{
      // 执行 Placement 对应操作
      commitPlacement(finishedWork);
      // 执行完 Placement 对应操作后，移除 Placement flag
      finishedWork.flags &= ~Placement;
      
      // 执行 Update 对应操作
      const current = finishedWork.alternate;
      commitWork(current, finishedWork);
      break;
    }
    // ...
  }
}
```

可以看出，Placement flag 对应的操作方法为 `commitPlacement()` ，代码如下：

```js
function commitPlacement(finishedWork){
  // 获取 Host 类型的祖先 FiberNode
  const parentFiber = getHostParentFiber(finishedWork);
  
  // 省略根据 parentFiber 获取对应 DOM 元素的逻辑
  
  let parent;
  
  // 目标 DOM 元素会插入至 before 左边
  const before = getHostSibling(finishedWork);
  
  // 省略分支逻辑
  // 执行插入或移动操作
  insertOrAppendPlacementNode(finishedWork, before, parent);
}
```

整个 `commitPlacement()` 方法执行流程可以分为三个步骤：
- 从当前 FiberNode 向上遍历，获取第一个类型为 HostComponent、HostRoot、HostPortal 三者之一的祖先 FiberNode，其对应的 DOM 元素是执行 DOM 操作的目标元素的父级 DOM 元素；
- 获取用于执行 `parent.insertBefore(child, before)` 方法的“before 对应的 DOM 元素”；
- 执行 `parentNode.insertBefore()` 方法（存在 before）或者`parentNode.appendChild()` 方法（不存在 before）

对于“还没有插入的 DOM 元素”（对应的就是 mount 场景），insertBefore 会将目标 DOM 元素插入到 before 之前，appendChild 会将目标 DOM 元素作为父 DOM 元素的最后一个子元素插入。

对于“UI 中已经存在的 DOM 元素”（对应 update 场景），insertBefore 会将目标 DOM 元素移动到 before 之前，appendChild 会将目标 DOM 元素移动到同级最后。

因此，这就是为什么 React 中，插入和移动所对应的 flag 都是 Placement flag 的原因。

3、更新 DOM 元素

更新 DOM 元素，一个主要的工作就是更新对应的属性，执行的方法为 `commitWork()` ，相关代码如下：

```js hl:11,15
function commitWork(current, finishedWork){
  switch(finishedWork.tag){
    // 省略其他类型处理逻辑
    case HostComponent:{
      const instance = finishedWork.stateNode;
      if(instance != null){
        const newProps = finishedWork.memoizedProps;
        const oldProps = current !== null ? current.memoizedProps : newProps;
        const type = finishedWork.type;

        const updatePayload = finishedWork.updateQueue;
        finishedWork.updateQueue = null;
        if(updatePayload !== null){
          // 存在变化的属性
          commitUpdate(instance, updatePayload, type, oldProps, newProps, finishedWork);
        }
      }
      return;
    }
  }
}
```

之前提到过，变化的属性会以 key、value 相邻的形式保存在 FiberNode.updateQueue，最终 FiberNode.updateQueue 里面所保存的要变化的属性会在名为 `updateDOMProperties()` 的方法中被遍历，然后进行处理。
该方法主要处理下面 4 种数据：
- style 属性变化
- innerHTML
- 直接文本节点变化
- 其他元素属性

相关代码如下：

```js
function updateDOMProperties(domElement, updatePayload, wasCustomComponentTag, isCustomComponentTag){
  for(let i=0;i< updatePayload.length; i+=2){
    const propKey = updatePayload[i];
    const propValue = updatePayload[i+1];
    if(propKey === STYLE){
      // 处理 style
      setValueForStyle(domElement, propValue);
    } else if(propKey === DANGEROUSLY_SET_INNER_HTML){
      // 处理 innerHTML
      setInnerHTML(domElement, propValue);
    } else if(propKey === CHILDREN){
      // 处理直接的文本节点
      setTextContent(domElement, propValue);
    } else {
      // 处理其他元素
      setValueForProperty(domElement, propKey, propValue, isCustomComponentTag);
    }
  }
}
```

当 Mutation 阶段的主要工作完成后，再进入 Layout 阶段之前，会执行如下的代码来完成 FiberTree 的切换：

```js
root.current = finishedWork;
```

## Layout 阶段

有关 DOM 的操作在 Mutation 阶段就已经结束了。

在 Layout 阶段，主要的工作集中在 `commitLayoutEffectsOnFiber()` 方法内部，这个方法内部会针对不同类型的 FiberNode 执行不同的操作：
- 对于 ClassComponent：该阶段会执行 `componentDidMount` / `componentDidUpdate` 方法；
- ==对于 FunctionComponent：该阶段会执行 useLayoutEffect 的回调；==
