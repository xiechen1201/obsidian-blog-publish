---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/35 Update/","dg-note-properties":{}}
---

在 React 中有很多触发更新的方法，例如：
- ReactDOM.createRoot
- this.setState
- this.forceUpdate
- useState dispatcher
- useReducer dispatcher

虽然这些方法执行的场景不同，但是都可以接入同样的更新流程，因为它们使用同一种数据结构来表示更新，这种数据结构就是 Update。

我们先从使用角度理解：

```jsx
function Counter(){
 const [num,setNum] = useState(0)

 return (
   <button onClick={()=>{
      setNum(1)
   }}>
      {num}
   </button>
 )

}
```

当执行 `setNum(1)` 的时候，React 不会立即重新渲染，因为 React 有调度系统。React 会先创建一个更新任务。

如果调用多次：

```js
setNum(1)
setNum(2)
setNum(3)
```

就需要多个 更新任务，这些更新任务需要进行保存，这个就是 UpdateQueue。

整体关系：

```md
用户操作

   |
   ↓

setState / useState

   |
   ↓

创建 Update

   |
   ↓

放入 updateQueue

   |
   ↓

render阶段计算最新state

   |
   ↓

commit更新DOM
```

## Update 数据结构

在 React 中，更新实际上是存在优先级的，其心智模型可以类比为“代码版本管理工具”。

例如，假如我们正在开发一个软件，当前软件处于正常的迭代中，拥有 A、B、C 三个正常需求，此时突然来了一个紧急的线上 Bug，整体流程如下：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-07-063812.png)
为了修复这个线上 Bug，你需要先完成需求 A、B、C，之后才能进行 D 的修复，这样的设计显然是不合理的。

在代码版本管理工具中，再出现紧急 Bug 需要修复时，可以先暂存当前分支的修改，在 master 分支修复 D 并紧急上线：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-07-064157.png)
当 Bug 修复完成后，再正常地迭代 A、B、C 需求，之后的迭代会基于 D 这个版本：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-07-064357.png)
并发更新的 React 也拥有相似的能力，==不同的 update 是有不同的优先级，高优先级的 update 能够中断低优先级的 update，当高优先级的 update 完成更新之后，后续的低优先级更新会在高优先级 update 更新后的 state 的基础上再来进行更新。==

接下来我们来看一下 Update 的一个数据结构。

前面提到，在 React 中有不同的触发更新的方法，不同的方法实际上对应不同的组件：
- ReactDOM.createRoot 对应 HostRoot
- this.setState 对应 ClassComponent
- this.forceUpdate 对应 ClassComponent
- useState dispatcher 对应 FunctionComponent
- useReducer dispatcher 对应 FunctionComponent

不同的组件类型，所对应的 Update 的数据结构是不同的。

HostRoot 和 ClassComponent 对应的 Update 数据结构如下：

```js hl:4,6,7
function createUpdate(eventTime, lane){
  const update = {
    eventTime,
    lane,
	// 区分触发更新的场景
    tag: UpdateState,
    payload: null,
    // UI 渲染后触发的回调函数
    callback: null,
    next: null,
  };
  return update;
}
```

在上面的 Update 数据结构中，tag 字段是用于区分触发更新的场景的，选项包括：
- ReplaceState：代表旧版 `replaceState` 的更新类型，在现代 React 的常规使用中基本不会出现；
- UpdateState：默认情况，通过 `root.render(...)` 或者 `this.setState` 触发更新；
- CaptureUpdate：代表发生错误的情况下在 ClassComponent 或者 HostRoot 中触发更新（例如通过 `getDerivedStateFromError()` 方法）；
- ForceUpdate：代表通过 `this.forceUpdate` 触发更新；

接下来看一下 FunctionComponent 所对应的 Update 数据结构：

```js hl:2,3
const update = {
  lane,
  action,
  // 优化策略相关字段
  hasEagerState: false,
  eagerState: null,
  next: null
}
```

上面的数据结构中，有 `hasEagerState` 和 `eagerState` 这两个字段，它们和 React 内部性能优化策略（eagerState 策略）相关。

在 Update 数据结构中，有三个问题是需要考虑的：
- 更新所承载的更新内容是什么

对于 HostRoot 以及类组件来说，承载更新内容的字段为 payload 字段：

```js
// HostRoot
ReactDOM.createRoot(rootEle).render(<App/>);                                   
// 对应 update
{
	payload : {
    // HostRoot 对应的 jsx，也就是 <App/> 对应的 jsx
  	element                                  
  },
  // 省略其他字段
}

// ClassComponent 情况1
this.setState({num : 1})
// 对应 update
{
  payload : {
    num: 1
  },
  // 省略其他字段
}

// ClassComponent 情况2
this.setState(prevState => ({ num: prevState.num + 1 }))
// 对应 update
{
  payload: prevState => ({ num: prevState.num + 1 }),
  // 省略其他字段
}
```

对于函数组件来讲，承载更新内容的字段为 action 字段：

```js
// FC 使用 useState 情况1
updateNum(1);
// 对应 update
{
  action : 1,
  // 省略其他字段
}

// FC 使用 useState 情况2
updateNum(num => num + 1);
// 对应 update
{
  action : num => num + 1,
  // 省略其他字段
}
```

- 更新的紧急程度：紧急程度是由 lane 字段所表示的；
- 更新之间的顺序：通过 next 字段来指向下一个 update，从而形成一个链表；

## UpdateQueue

==上面介绍的 update 是计算 state 的最小单位，updateQueue 是由 update 组成的一个链表，==updateQueue 的数据结构如下：

```js
const updateQueue = {
  baseState: null,
  firstBaseUpdate: null,
  lastBaseUpdate: null,
  shared: {
    pending: null
  }
}
```

- baseState：参与计算的初始 state，update 基于该 state 计算新的 state，可以类比为心智模型中的 master 分支；
- firstBaseUpdate 和 lastBaseUpdate：指向先前因优先级不足而被跳过、需在后续渲染中继续处理的 base update 链表。链表头部为 firstBaseUpdate，链表尾部为 lastBaseUpdate；
- shared.pending：指向由新产生的 update 组成的单向环状链表。计算 state 时，该环状链表会被拆分并拼接在 lastBaseUpdate 后面；

举例说明，例如当前有一个 FiberNode 刚经历完 commit 阶段的渲染，该 FiberNode 上面有两个“由于低优先级，导致在上一轮 render 阶段并没有被处理的 update”，假设这两个 update 分别名为 u0 和 u1。

```js
fiber.updateQueue.firstBaseUpdate = u0;
fiber.updateQueue.lastBaseUpdate = u1;
u0.next = u1;
```

那么假设在当前的 FiberNode 上面，我们又触发了两次更新，分别产生了两个 update（u2 和 u3），新产生的 update 就会形成一个环状链表，shared.pending 就会指向这个环状链表，如下图所示：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-07-085309.png)
之后进入新一轮的 render，在该 FiberNode 的 beginWork 中，shared.pending 所指向的环状链表就会被拆分，拆分之后接入到 baseUpdate 链表后面：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-07-085521.png)
接下来就会遍历由 updateQueue.firstBaseUpdate 和 updateQueue.lastBaseUpdate 标识的 base update 链表，基于 updateQueue.baseState 来计算每个符合优先级条件的 update（这个过程类似 Array.prototype.reduce），最终计算出最新的 state，该 state 被称之为 memoizedState。

最后总结一下。整个 state 计算的流程可以分为两步：
- 将 shared.pending 所指向的环状链表进行拆分，并和 base update 链表进行拼接，形成新的链表；
- 遍历连接后的链表，根据本轮的 renderLanes 筛选符合优先级条件的 update，并计算 state；
