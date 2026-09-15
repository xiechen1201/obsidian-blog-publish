---
{"dg-publish":true,"permalink":"//web/4/react18/25-complete-work/","tags":["gardenEntry"],"dg-note-properties":{}}
---

`completeWork()` 属于“归”阶段。
和 `beginWork()` 类似，`completeWork()` 也会会根据 wip.tag 区分对待，流程上包括两个步骤：
- 创建元素或者标记元素更新；
- flags 冒泡；

整体流程图如下：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-01-060445.png)
## mount 阶段

（* 使用 wip.tag = HostComponent 为示例）

在 mount 流程中，首先通过 `createInstance()` 创建 FiberNode 所对应的 DOM 元素：

```js hl:8
function createInstance(type, props, rootContainerInstance, hostContext, internalInstanceHandle){
  //...
  if(typeof props.children === 'string' || typeof props.chidlren === 'number'){
    // children 为 string 或者 number 时做一些特殊处理
  }
  
  // 创建 DOM 元素
  const domElement = createElement(type, props, rootContainerInstance, parentNamespace);
  
  //...
  return domElement;
}
```

接下来会执行 `appendAllChildren`，该方法的作用就是将下一层 DOM 元素插入到 `createInstance()` 所创建的 DOM 元素中，具体逻辑如下：
- 从当前 FiberNode 向下遍历，将遍历到的第一层 DOM 元素类型（HostComponent、HostText）通过 appendChild 插入到 parent 末尾；
- 对兄弟 FiberNode 执行步骤 1；
- 如果没有兄弟 FiberNode，则对父 FiberNode 的兄弟执行步骤 1；
- 当遍历流程回到最初执行步骤 1 所在层或者 parent 所在层终止；

相关代码如下：

```js
appendAllChildren = function(parent, workInProgress, ...){
  let node = workInProgress.child;
  
  while(node !== null){
    // 步骤 1，向下遍历，对第一层 DOM 元素执行 appendChild
    if(node.tag === HostComponent || node.tag === HostText){
      // 对 HostComponent、HostText 执行 appendChild
      appendInitialChild(parent, node.stateNode);
    } else if(node.child !== null) {
      // 继续向下遍历，直到找到第一层 DOM 元素类型
      node.child.return = node;
      node = node.child;
      continue;
    }
    // 终止情况 1: 遍历到 parent 对应的 FiberNode
    if(node === workInProgress) {
      return;
    }
    // 如果没有兄弟 FiberNode，则向父 FiberNode 遍历
    while(node.sibling === null){
      // 终止情况 2: 回到最初执行步骤 1 所在层
      if(node.return === null || node.return === workInProgress) {
        return;
      }
      node = node.return
    }
    // 对兄弟 FiberNode 执行步骤 1
    node.sibling.return = node.return;
    node = node.sibling;
  }
}
```

`appendAllChildren()` 方法实际上就是在处理下一层级 DOM，而且在 `appendAllChildren()` 里面遍历过程会更加复杂一些，会多一些判断，因为 FiberNode 最终形成的 FiberTree 的层次和最终 DOMTree 的层级可能有区别：

```jsx
function World(){
  return <span>World</span>
}

<div>
	Hello
	<World/>
</div>
```

上面的示例中，如果从 FiberNode 的角度来看，Hello 和 World 是同层级的。但是如果从 DOM 元素的角度来看，Hello 和 <span /> 是同层级的。因此从 FiberNode 中查找同层级的 DOM 元素的时候，经常会涉及到跨 FiberNode 层级进行查找。

接下来 `completeWork()` 会执行 `finalizeInitialChildren()` 方法完成属性的初始化，主要包含以下几类属性：
- styles，对应的方法为 `setValueForStyles()`方法；
- innerHTML，对应 `setInnerHTML()`方法；
- 文本类型 children，对应 `setTextContent()`方法；
- 不会再在 DOM 中冒泡的事件，包括 cancel、close、invalid、load、scroll、toggle，对应点的是 `listenToNonDelegatedEvent()`方法；
- 其他属性，对应 `setValueForProperty()` 方法；

`finalizeInitialChildren()` 方法执行完成后，最后进行 flags 的冒泡；

总结一下，`completeWork()` 在 mount 阶段执行的流程：
1. 根据 wip.tag 进行不同分支处理；
2. 根据 `current !== null` 区分是 mount 还是 update；
3. 对应 HostComponent，首先要执行 `createInstance()` 方法来创建对应的 DOM 元素；
4. 执行 `appendChildren()` 将下一级 DOM 元素挂载到 `createInstance()` 创建的 DOM 元素下；
5. 执行 `finalizeInitialChildren()` 方法完成属性初始化；
6. 执行 `bubbleProperties()` 完成 flags 冒泡；

## update 阶段

