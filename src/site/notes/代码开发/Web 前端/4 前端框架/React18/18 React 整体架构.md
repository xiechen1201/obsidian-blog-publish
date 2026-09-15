---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/18 React 整体架构/","dg-note-properties":{}}
---

React15 以及之前的架构称之为 Stack 架构，从 React16 开始 React 重构了整体的架构，新的架构称之为 Fiber 架构。新的架构相比旧的架构有一个最大的特点就是能够实现时间切片。

## 旧架构的问题

首先需要想一想，在 Web 中哪些情况会导致页面无法快速响应呢？
总结起来，实际上也就是两大场景：
- 需要执行大量计算或者设备本身性能不足的情况下，页面就会出现掉帧、卡顿的现象，这个情况本质上是 CPU 的瓶颈；
- 进行 I/O 的时候，需要等待数据返回后再进行后续操作，等待的过程中无法快速响应，这种情况实际上来自于 I/O 的瓶颈；

### CPU 瓶颈

我们在浏览网页的时候，这个网页实际上是浏览器绘制出来的，就像一个画家在画画一样。网页中往往又存在一些动起来的效果，例如轮播图、百叶窗之类的，本质上就是浏览器不停的在绘制。

目前大多数设备的刷新率为 60FPS，意味着每 1s 就需要绘制 60 次，1000ms / 60 = 16.66ms，也就说每隔 16.66ms 就需要绘制一帧。

浏览器在绘制页面的时候，实际上有很多的工作需要处理（[[代码开发/Web 前端/5 浏览器与网络/浏览器/01 浏览器的渲染流程\|01 浏览器的渲染流程]]）：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-27-060044.png)
上图中的任务称之为“渲染流水线”，每次执行流水线的时候，大致是需要如上的一些步骤，但并不是说每一次所有的任务都需要全部执行上面的流程。
例如：
- 通过 JS 或者 CSS 修改 DOM 元素的几何属性（如长度、宽度）时，会触发完整的渲染流水线，这种情况称之为重排；
- 当修改的属性不涉及几何属性（如字体、颜色）时，会省略流水线中 Layout、Layer 的过程，这种情况称为「重绘」；
- 当修改不涉及重排、重绘的属性（如 transform）时，会省略流水线中 Layout、Layer、Print 过程，仅执行合成线程的绘制工作，这种情况称之为「合成」；
按照性能高低进行排序：合成 > 重绘 > 重排。

==上面提到了，浏览器绘制的频率是 16.66ms 一帧，但是执行 JS 与渲染流水线实际上是在同一个线程中执行的，也就意味着如果 JS 执行时间太长，不能够及时的渲染下一帧，就会造成页面的掉帧现象，页面就会变得卡顿。==

在 React 16 之前，React 的更新过程是同步且不可中断的。
当组件发生更新时，React 需要重新计算组件树对应的虚拟 DOM，并进行 Diff。虽然虚拟 DOM 减少了真实 DOM 操作的成本，但在大型应用中，一次完整的更新任务仍可能占用较长的 JavaScript 执行时间。由于 JS 执行与浏览器渲染共享同一个主线程，长时间运行的更新任务会阻塞浏览器绘制，导致帧率下降，最终表现为动画不流畅、交互延迟和页面卡顿。

假如有下面的 DOM 层级结构：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-23-072638.png)
转化为虚拟 DOM 对象结构大致如下：
```js
{
  type : "div",
  props : {
    id : "test",
    children : [
      {
        type : "h1",
        props : {
          children : "This is a title"
        }
      }
      {
        type : "p",
        props : {
          children : "This is a paragraph"
        }
      },{
        type : "ul",
        props : {
          children : [{
            type : "li",
            props : {
              children : "apple"
            }
          },{
            type : "li",
            props : {
              children : "banana"
            }
          },{
            type : "li",
            props : {
              children : "pear"
            }
          }]
        }
      }
    ]
  }
}
```

在 React16 版本之前，进行两颗虚拟 DOM 树对比的时候，需要遍历上面的结构，这个时候只能使用递归，而且这种递归是不能够进行打断的，只能一条路走到黑，从而导致了 JS 执行时间过长。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-27-070133.png)
这样的架构模式官方称之为 Stack 架构模式，因为采用的是递归，会不停的开启新的函数栈。

### I/O 瓶颈

对前端来说，主要的 I/O 就是网络延迟。
网络延迟是一种客观存在的现象，如何减少网络延迟对用户的影响呢？React 团队给出的方案是：将人机交互的研究成果整合到 UI 中。

用户对卡顿的感知是不一样的，例如输入框哪怕轻微的延迟，用户也会认为卡顿。假如一个 List 列表，加上 Loading 的动画，哪怕加载很长时间，用户也不会觉得卡顿。

对于 React 来说，所有的操作都来自于 State 的变化导致重新渲染，我们只需要针对不同的操作赋予不同的优先级即可。
具体来说包含以下三个方面：
- 为不同操作造成的“状态变化”赋予不同的优先级；
- 所有优先级统一调度，优先处理高优先级的更新；
- 如果更新正在进行（进入虚拟 DOM 相关工作），此时有更高优先级的更新产生的话，中断当前的更新，优先处理高优先级的更新；
要实现上面的三个点，就需要 React 底层能实现：
- 能够调度优先级的调度器；
- 调度器对应的调度算法；
- 支持可中断的虚拟 DOM 的实现；

所以，不管是解决 CPU 瓶颈还是 I/O 的瓶颈，底层的诉求都是实现时间切片 time slice。

## 新架构的解决思路

### 解决 CPU 瓶颈

从 React16 开始，官方团队正式引用了 Fiber 的概念，这是一种通过「链表」来描述 UI 的方式，<u>本质上也可以看作是虚拟 DOM 的实现。</u>

可以把链表想象成“手里拿着下一个人在哪里的纸条”。例如：

```js
const A = {
  value: "A",
  next: B
};

const B = {
  value: "B",
  next: C
};

const C = {
  value: "C",
  next: null
};
```

链表的核心不是它们挨着放，而是通过指针/引用连接起来，它特别适合关系经常变化的数据。

```md
目前：
A → B → C

在 A 和 B 中间插入 X：
[A, B, C] → [A, X, B, C]

本质上就是：
A.next = X
X.next = B
```

Fiber 本质上也是一个对象，但是和之前的 React 元素不同的地方在于对象之间使用的是链表结构进行连接，`child` 指向子元素，`sibling` 指向兄弟元素，`return` 指向父元素。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-24-032509.png)
==使用链表结构有一个很大好处就是在进行整棵树对比（reconcile）计算的时候，这个过程是可以中断的。==

当发现一帧时间已经不够的时候，不能继续执行 JS，需要优先渲染下一帧的时候，这个时候就可以中断 JS 的执行，优先渲染下一帧的画面。渲染完成后再接着回来完成上一次没有执行完的 JS 计算。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-27-070226.png)
官方还提供了一个 Stack 结构和 Fiber 架构的对比示例：

```cardlink
url: https://claudiopro.github.io/react-fiber-vs-stack-demo/
title: "Fiber vs Stack Demo"
host: claudiopro.github.io
```

下面是 React 源码中创建 Fiber 对象的相关代码：

```js hl:14-19
const createFiber = function (tag, pendingProps, key, mode) {
  // 创建 fiber 节点的实例对象
  return new FiberNode(tag, pendingProps, key, mode);
};

function FiberNode(tag, pendingProps, key, mode) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null; // 映射真实 DOM

  // Fiber
  // 上下、前后 fiber 通过链表的形式进行关联
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;
  this.refCleanup = null;
  // 和 hook 相关
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;

  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  this.alternate = null;
  // ...
}
```

### 解决 I/O 瓶颈

从 React16 开始引入 Scheduler 调度器，用来调度优先级不同的任务。

在 React16 之前架构中有下面两个架构组件：
- Reconciler 协调器：vdom 的实现，根据 State 的变化计算出 UI 的变化；
- Renderer 渲染器：负责将 UI 的变化渲染到宿主环境；

从 React16 开始，多了一个组件：
- Scheduler 调度器：调度任务的优先级，高优先级的任务会优先进入到 Reconciler；
- Reconciler 协调器（同上）；
-  Renderer 渲染器（同上）；

新的架构中，Reconciler 的更新流程也从之前的递归变成了可中断的循环过程。

```js
function workLoopConcurrent(){
  // 如果还有任务，并且时间切片还有剩余的时间
  while(workInProgress !== null && !shouldYield()){
    // 执行工作单元
    performUnitOfWork(workInProgress);
  }
}

function shouldYield(){
  // 当前时间是否大于过期时间
  // 其中 deadline = getCurrentTime() + yieldInterval
  // yieldInterval 为调度器预设的时间间隔，默认为 5ms
  return getCurrentTime() >= deadline;
}
```

每次循环都会调用 `shouldYield()` 方法来判断当前的时间切片是否还有足够的剩余时间，如果没有足够的剩余时间，就暂停 Recociler 的执行，将主线程还给渲染流水线，进行下一帧的渲染操作，渲染工作完成后，在等待下一个宏任务进行后续代码的执行。