---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/28 lane 模型/","dg-note-properties":{}}
---

## React 和 Scheduler 优先级的介绍

之前我们介绍过 Scheduler，React 团队打算将这个 Scheduler 进行独立的发布。

在 React 内部还会有一个粒度更细的优先级算法，这个就是 lane 模型。

接下来看一下两套优先级模型的一个转换。

在 Scheduler 内部，拥有 5 种优先级：

```js
export const NoPriority = 0; // 无优先级
export const ImmediatePriority = 1;
export const UserBlockingPriority = 2;
export const NormalPriority = 3;
export const LowPriority = 4;
export const IdlePriority = 5;
```

作为一个独立的包，需要考虑到通用性，Scheduler 和 React 的优先级并不互通，在 React 的内部，有 4 种优先级：

```js
export const DiscreteEventPriority: EventPriority = SyncLane;
export const ContinuousEventPriority: EventPriority = InputContinuousLane;
export const DefaultEventPriority: EventPriority = DefaultLane;
export const IdleEventPriority: EventPriority = IdleLane;
```

由于 React 中不同的交互对应的事件回调中产生的 update 会有不同的优先级，因此优先级和事件有关，所以 React 内部的优先级也被称为 EventPriority，各个等级的含义如下：
- DiscreteEventPriority：对应离散事件优先级，例如 click、input、focus、blur、touchstart 等事件都是离散触发的；
- ContinuousEventPriority：对应连续事件的优先级，例如 drag、mousemove、scroll、touchmove 等事件都是连续触发的；
- DefaultEventPriority：对应默认的优先级，例如通过计时器周期性触发更新，这种情况下产生的 update 不属于交互产生 update，所以优先级是默认的优先级；
- IdleEventPriority：对应空闲情况的优先级；

上面的代码中，我们可以看到不同的级别 EventPriority 对应不同的 lane。

既然 React 和 Scheduler 的优先级不互通，那么这里就涉及到一个转换的问题，这里分为：
- React 优先级转换为 Scheduler 的优先级；
- Scheduler 的优先级转换为 React 的优先级；

## React 优先级转换为 Scheduler 的优先级

整体会经历两次转换：
- 首先是将 lanes 转换为 EventPriority，涉及到的方法如下：

```js
export function lanesToEventPriority(lanes: Lanes): EventPriority {
  // getHighestPriorityLane 方法用于分离出优先级最高的 lane
  const lane = getHighestPriorityLane(lanes);
  
  if (!isHigherEventPriority(DiscreteEventPriority, lane)) {
    return DiscreteEventPriority;
  }
  if (!isHigherEventPriority(ContinuousEventPriority, lane)) {
    return ContinuousEventPriority;
  }
  if (includesNonIdleWork(lane)) {
    return DefaultEventPriority;
  }
  return IdleEventPriority;
}
```

- 将 EventPriority 转换为 Scheduler 的优先级，方法如下：

```js
// ...
let schedulerPriorityLevel;
switch (lanesToEventPriority(nextLanes)) {
  case DiscreteEventPriority:
    schedulerPriorityLevel = ImmediateSchedulerPriority;
    break;
  case ContinuousEventPriority:
    schedulerPriorityLevel = UserBlockingSchedulerPriority;
    break;
  case DefaultEventPriority:
    schedulerPriorityLevel = NormalSchedulerPriority;
    break;
  case IdleEventPriority:
    schedulerPriorityLevel = IdleSchedulerPriority;
    break;
  default:
    schedulerPriorityLevel = NormalSchedulerPriority;
    break;
}
// ...
```

例如现在有一个点击事件，在 `onClick` 中对应有一个回调函数来触发更新，这个更新属于 DiscreteEventPriority，经过上面的两套转换规则进行转换后，最终对应的 Scheduler 优先级是 ImmediateSchedulerPriority。

## Scheduler 的优先级转换为 React 的优先级

转换的代码如下：

```js
const schedulerPriority = getCurrentSchedulerPriorityLevel();
switch (schedulerPriority) {
  case ImmediateSchedulerPriority:
    return DiscreteEventPriority;
  case UserBlockingSchedulerPriority:
    return ContinuousEventPriority;
  case NormalSchedulerPriority:
  case LowSchedulerPriority:
    return DefaultEventPriority;
  case IdleSchedulerPriority:
    return IdleEventPriority;
  default:
    return DefaultEventPriority;
}
```

这里就会涉及到一个问题，如果在同一时间可能存在很多的更新，究竟先去更新哪一个呢？
- 从众多的有优先级的 update 中选出一个优先级最高的；
- 表达批的概念；

React 在批处理的表达方式上实际上经历了两次迭代：
- 基于 expirationTime 的算法
- 基于 lane 的算法

## expirationTime 模型

React 早期采用的就是 expirationTime 算法，这一点和 Scheduler 里面的设计是一致的。

在 Scheduler 中，设计了 5 种优先级，不同的优先级会对应不同的 timeout，最终会对应不同的 expirationTime，然后 task 根据 expirationTime 进行任务的排序。

早期的 React 延续了这种设计，update 的优先级与触发事件的当前时间，以及优先级对应的延迟时间有关，这样的算法实际上是比较简单易懂的。每当进入 schedule 的时候，就会选出优先级最高的 update 进行一个调度。

但是这种算法在表示“批”的概念上不够灵活。

在基于 expirationTime 模型的算法中，有如下的表达：

```js
const isUpdateIncludedInBatch = priorityOfUpdate >= priorityOfBatch;
```

priorityOfUpdate 表示的是当前 update 的优先级，priorityOfBatch 代表的是批对应的优先级下限。也就是说，当前的 update 只要大于等于 priorityOfBatch 就会被划分为同一批。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-16-032346.png)
但是此时就会存在一个问题，如何将某一范围的某几个优先级划分为同一批呢？
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-16-032601.png)
在上图中，我们想要将 u1、u2、u3 和 u4 划分为同一批，但是之前的 expirationTime 模型是无法做到的。

究其原因，是因为 expirationTime 模型优先级算法耦合了“优先级”和“批”的概念，限制了模型的表达能力。
优先级算法的本质是为 update 进行一个排序，但是 expirationTime 模型在完成排序的同时还默认划定了“批”。

## lane 模型

所以，基于上面的原因，React 引入了 lane 模型。

不管引入什么模型，都要保证以下两个问题得到解决：
- 以优先级为依据，对 update 进行排序；
- 表达批的概念；

针对第一个问题，lane 模型中设置了很多 lane，每一个 lane 实际上是一个二进制数，通过二进制来表达优先级，越低的位代表越高的优先级，例如：

```js
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
export const InputContinuousLane: Lane = /*             */ 0b0000000000000000000000000000100;
export const DefaultLane: Lane = /*                     */ 0b0000000000000000000000000010000;
export const IdleLane: Lane = /*                        */ 0b0100000000000000000000000000000;
export const OffscreenLane: Lane = /*                   */ 0b1000000000000000000000000000000;
```

在上面的代码中，SyncLane 是最高优先级，OffscreenLane 是最低优先级。

对于第二个问题，lane 模型能够非常灵活的表达批的概念：

```js
// 要使用的批
let batch = 0;
// laneA 和 laneB 是不相邻的优先级
const laneA = 0b0000000000000000000000001000000;
const laneB = 0b0000000000000000000000000000001;
// 将 laneA 纳入批中
batch |= laneA;
// 将 laneB 纳入批中
batch |= laneB;
```
