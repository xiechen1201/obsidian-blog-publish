---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/20 Fiber 双缓冲/","dg-note-properties":{}}
---

## 对 Fiber 的理解

可以从三个维度来理解 Fiber：
- 一种架构，称之为 Fiber 架构；
- 一种数据类型；
- 动态的工作单元；

1、一种架构

React 从 16 版本开始重构了整个架构，引入了 Fiber，新的架构模式也称为 Fiber 架构，各个 FiberNode 使用链表的形式进行连接。

```js
function FiberNode(tag, pendingProps, key, mode) {
  // ...

  // 周围的 Fiber Node 通过链表的形式进行关联
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  // ...
}
```

2、一种数据类型

Fiber 本质上也是一个对象，是在前 React 元素的基础上的一种升级版本，每个 FiberNode 对象里面会包含 React 元素的类型、周围连接的 FiberNode 以及 DOM 相关信息：

```js
function FiberNode(tag, pendingProps, key, mode) {
  // 类型
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null; // 映射真实 DOM

  // ...
}
```

3、动态的工作单元

每个 FiberNode 中保存了本次更新中该 React 元素变化的数据，还包括要执行的工作（增、删、更新）以及副作用信息：

```js
function FiberNode(tag, pendingProps, key, mode) {
  // ...

  // 副作用相关
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;
	// 与调度优先级有关  
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // ...
}
```

> [!question]
> 为什么指向父 FiberNode 的字段是 return 而不是 parent？
> 因为作为一个动态的工作单元，return 指的是 FiberNode 执行完 `completeWork()` 后返回的下一个 FiberNode，这里有一个返回的动作，因此通过 return 来指代父 FiberNode。

## Fiber 双缓冲

Fiber 架构中的双缓冲工作原理类似于显卡的工作原理。

> [!info] 
> 双缓冲要解决的问题是：不要一边修改用户正在看的东西，一边把半成品展示给用户。
> 假如没有双缓冲，屏幕中只有一块画布，用户正在观看画面。然后下一帧画面来了，显卡直接在当前画面中进行修改，用户可能看到修改一半的画面。
> 双缓冲的办法非常直接，前缓冲区存放用户正在观看的画面，后缓冲区偷偷的绘制下一帧画面，等到新画面完全准备好后，前后缓冲区进行交换（前缓冲区变成后缓冲区，后缓冲区变成前缓冲区），用户看到的就是新的、完整的画面。

Fiber 架构同样使用到这个技术，在 Fiber 架构中同时存在两颗 FiberTree，一颗是真实 UI 对应的 FiberTree（类比为显卡的前缓冲区），另外一颗是是在内存中构建的 FiberTree（类比为显卡的后缓冲区）。

在 React 的源码中，很多的方法都接收两棵 FiberTree：

```js
function cloneChildFibers(current, workInProgress){
  // ...
}
```

current 指的就是前缓冲区的 FiberNode，workInProgress 指的就是后缓冲区的 FiberNode。
两个 FiberNode 会通过 `alternate` 属性相互指认：

```js
current.alternate = workInProgress;
workInProgress.alternate = current;
```

下面我们从首次渲染（mount）和更新（update）两个阶段来看一下 FiberTree 的形成以及双缓冲机制。

### mount 阶段

==首先最顶层有一个 FiberNode，称之为 FiberRootNode，该 FiberNode 会有一些自己的任务：==
- Current FiberTree 和 WorkInProgress FiberTree 之间的切换；
- 应用中的过期时间；
- 应用的任务调度信息；

假如现在存在这样的一段结构：

```html
<body>
  <div id="root"></div>
</body>
```
```js
function App(){
  const [num, add] = useState(0);
  return (
  	<p onClick={() => add(num + 1)}>{num}</p>
  );
}
const rootElement = document.getElementById("root");
ReactDOM.createRoot(rootElement).render(<App />);
```

当执行 `ReactDOM.createRoot(rootElement)` 的时候会创建如下的结构：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-24-071516.png)
此时会有一个 HostRootFiber，FiberRootNode 通过 current 来指向 HostRootFiber。

此时页面还没有 p 元素。

接下来进入到 mount 流程（也就是执行 `.render()` 的时候），这个流程会基于每个 React 元素以深度优先的原则依次生成 WorkInProgress FiberNode，并且每一个 WorkInProgress FiberNode 会连接起来，如下图：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-24-072421.png)
生成的 WorkInProgress FiberTree 里面的每一个 FiberNode 会和 current FiberTree里面的 FiberNode 进行关联，关联的方式就是通过 alternate。但是目前 current FiberTree 里面只有一个 HostRootFiber，因此就只有这个 HostRootFiber 进行了 alternate 的关联。

当 WorkInProgress FiberTree 生成完毕后，也就意味着 render 阶段完成了，此时 FiberRootNode 就会被传递给 Renderer 渲染器，接下来就是进行渲染工作。

渲染工作完成后，浏览器中就显示了对应的 UI，此时 FiberRootNode.current 就会指向这颗 WorkInProgress FiberTree，曾经的 WorkInProgress FiberTree 它就会变成 current FiberTree，完成了双缓冲的工作：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-24-072953.png)
> [!info]
> 为什么第一次的时候 current 只有一个 HostRootFiber？
> 因为 React 首次运行的是还没有“旧版本”，此时还没有处理 JSX，只有一个基准的根节点 HostRootFiber。当执行 `.render(<App />)` 的时候才开始真正的渲染内容，也就是从 WorkInProgress Fiber Tree 开始。

### update 阶段

假如点击 p 元素会触发更新，这个操作就会开启 update 流程，此时就会生成一颗新的 WorkInProgress FiberTree，流程和之前一致。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-24-073250.png)
新的 WorkInProgress FiberTree 里面的每一个 FiberNode 和 current FiberTree 的每一个 FiberNode 通过 alternate 属性进行关联。

当 WorkInProgress FiberTree 生成完毕后，就会经历和之前一样的流程，FiberRootNode 会被传递给 Renderer 进行渲染，此时的宿主环境所渲染出来的真实 UI 对应的是左边的 WorkInProgress FiberTree 所对应的 DOM 结构。`FiberRootNode.current` 就会指向左边的这颗树，右边的树再次称为了新的 WorkInProgress FiberTree。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-24-073639.png)
这个就是 Fiber 的双缓冲原理。

另外值得一提的是，开发者可以在一个页面中创建多个应用，例如：

```js
ReactDOM.createRoot(rootElement1).render(<App1 />);
ReactDOM.createRoot(rootElement2).render(<App2 />); 
ReactDOM.createRoot(rootElement3).render(<App3 />);  
```

上面的代码中我们创建了 3 个应用，此时就会存在 3 个 FiberRootNode，以及最多 6 颗 Fiber Tree 树。

AI 总结双缓冲过程图：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/ChatGPT_Image_2026%E5%B9%B49%E6%9C%887%E6%97%A5_16_51_13.png)