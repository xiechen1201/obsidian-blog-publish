---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/30 Hooks 原理/","dg-note-properties":{}}
---

## Hook 内部介绍

在 React 中针对 Hook 有三种策略，或者说三种类型的 dispatcher：
- HooksDispatcher：复杂初始化工作，让函数组件的一些初始化信息挂载到 Fiber 上面；

```js
/* 函数组件初始化用的 hooks */
const HooksDispatcherOnMount: Dispatcher = {
  readContext,
  ...
  useCallback: mountCallback,
  useEffect: mountEffect,
  useMemo: mountMemo,
  useReducer: mountReducer,
  useRef: mountRef,
  useState: mountState,
  ...
};
```

- HooksDispatcherOnUpdate：函数组件进行更新的时候，会执行这个对象对应的方法。此时 Fiber 上面存储了函数组件相关的信息，这些 Hook 需要做的就是去获取或者更新维护这些 Fiber 的信息；

```js
/* 函数组件更新用的 hooks */
const HooksDispatcherOnUpdate: Dispatcher = {
  readContext,
  ...
  useCallback: updateCallback,
  useContext: readContext,
  useEffect: updateEffect,
  useMemo: updateMemo,
  useReducer: updateReducer,
  useRef: updateRef,
  useState: updateState,
  ...
};
```

- ContextOnlyDispatcher：这个和报错相关，防止开发者在函数组件外部调用 Hook；

```js
/* 当hooks不是函数组件内部调用的时候，调用这个hooks对象下的hooks，所以报错。 */
export const ContextOnlyDispatcher: Dispatcher = {
  readContext,
  ...
  useCallback: throwInvalidHookError,
  useContext: throwInvalidHookError,
  useEffect: throwInvalidHookError,
  useMemo: throwInvalidHookError,
  useReducer: throwInvalidHookError,
  useRef: throwInvalidHookError,
  useState: throwInvalidHookError,
  ...
};
```

总结一下：
- mount 阶段：函数组件是进行初始化，那么此时调用的就是 mountXXX 对应的函数；
- update 阶段：函数组件进行状态更新，调用的就是 updateXXX 对应的函数；
- 其他场景下（报错）：此时调用的就是 throwInvalidError；

当 FC（函数组件）进入到 Render 流程的时候，首先会判断是初次渲染还是更新：

```js
if(current !== null && current.memoizedState !== null) {
  // 说明是 update
  ReactCurrentDispatcher.current = HooksDispatcherOnUpdate;
} else {
  // 说明是 mount
  ReactCurrentDispatcher.current = HooksDispatcherOnMount;
}
```

判断 mount 还是 update 之后，会给 ReactCurrentDispatcher.current 赋值对应的 dispatcher，因此赋值了不同的上下文对象，因此就可以根据不同上下文对象调用不同的方法。

例如有嵌套的 hook：

```js
useEffect(()=>{
  useState(0);
})
```

然后此时的上下文对象所指向 ContextOnlyDispatcher，最终执行的就是 throwInvalidHookError，抛出错误。

接下来我们来看一下 hoos 的数据结构：

```js
const hook = {
  memoizedState: null,
  baseState: null,
  baseQueue: null,
  queue: null,
  next: null
}
```

这里需要注意 `memoizedState` 属性，因为在 FiberNode 上也有这个属性，和 Hoos 对象上面的 `memoizedState` 存储的东西是不一样的：
- `FiberNode.memoizedState` 保存的是 Hook 链表里面的第一个链表；
- `hook.memoizedState` 保存的是某个 hook 自身的数据；

不同类型的 hook，`hook.memoizedState` 所存储的内容也是不同的：
- useState：对于 `const [state, updateState] = useState(initialState)`，memoizedState 保存的是 state 的值；
- useReducer：对于`const [state, dispatch] = useReducer(reducer, {})`，memoizedState 保存的是 state 的值；
- useEffect：对于 `useEffect( callback, [...deps])`，memoizedState 保存的是 callback、[...deps] 等数据；
- useRef：对于 `useRef(initialValue)`，memoizedState 保存的是 `{current: initialValue}` ；
- useMemo：对于 `useMemo( callback, [...deps] )` ，memoizedState 保存的是 [callback、[...deps]] 数据；
- useCallback：对于`useCallback( callback, [...deps] )`，memoizedState 保存的是 [callback、[...deps]] 数据；
有些 Hook 不需要 memoizedState 保存自身数据，例如 useContext。

## Hook 执行流程

当 FC 进入到 Render 阶段的时候，会被 `renderWithHooks()` 函数处理执行：

```js
export function renderWithHooks(current, workInProgress, Component, props, secondArg, nextRenderLanes) {
  renderLanes = nextRenderLanes;
  currentlyRenderingFiber = workInProgress;

  // 每一次执行函数组件之前，先清空状态 （用于存放hooks列表）
  workInProgress.memoizedState = null;
  // 清空状态（用于存放effect list）
  workInProgress.updateQueue = null;
  // ...

  // 判断组件是初始化流程还是更新流程
  // 如果初始化用 HooksDispatcherOnMount 对象
  // 如果更新用 HooksDispatcherOnUpdate 对象
  // 初始化对应的上下文对象，不同的上下文对象对应了一组不同的方法
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
      ? HooksDispatcherOnMount
      : HooksDispatcherOnUpdate;

  // 执行我们真正函数组件，所有的 hooks 将依次执行。
  let children = Component(props, secondArg);

  // ...

  // 判断环境
  finishRenderingHooks(current, workInProgress);
  return children;
}

function finishRenderingHooks(current, workInProgress) {
    // 防止 hooks 在函数组件外部调用，如果调用直接报错
    ReactCurrentDispatcher.current = ContextOnlyDispatcher;
    // ...
}

```

`renderWithHooks()` 会被每次函数组件触发时（mount、update），该方法就会清空 workInProgress 的 `memoizedState` 以及 `updateQueue` 。接下来判断组件是初始化还是更新，为 `ReactCurrentDispatcher.current` 肤质不同的上下文对象，之后调用 `Component()` 方法来执行函数组件，组件里面所写的 hook 就会依次执行。

下面以 useState 为案例看一下整个 Hook 的执行流程：

```jsx
function App(){
  const [count, setCount] = useState(0);
  return <div onClick={()=> setCount(count+1)}>{count}</div>
}
```

接下来就会根据目前是 mount 还是 update 调用不同的上下文里面对应的方法。

mount 阶段调用的是 `HooksDispatcherOnMount.mountState()`，相关代码如下：

```js
function mountState(initialState) {
  // 1. 拿到 hook 对象
  const hook = mountWorkInProgressHook();
  if (typeof initialState === "function") {
    initialState = initialState();
  }
  
  // 2. 初始化hook的属性
  // 2.1 设置 hook.memoizedState/hook.baseState
  hook.memoizedState = hook.baseState = initialState;
  const queue = {
    pending: null,
    lanes: NoLanes,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: initialState,
  };
  // 2.2 设置 hook.queue
  hook.queue = queue;
  
  // 2.3 设置 hook.dispatch
  const dispatch = (queue.dispatch = dispatchSetState.bind(
    null,
    currentlyRenderingFiber,
    queue
  ));
  
  // 3. 返回[当前状态, dispatch函数]
  return [hook.memoizedState, dispatch];
}
```

上面在执行 `mountState()` 的时候，首先调用了 `mountWorkInProgressHook()` 方法，这个方法就是创建一个 hook 对象，相关代码如下：

```js
function mountWorkInProgressHook() {
  const hook = {
    memoizedState: null, // Hook 自身维护的状态

    baseState: null,
    baseQueue: null,
    queue: null, // Hook 自身维护的更新队列

    next: null, // next 指向下一个 Hook
  };
  
  // 最终 hook 对象是要以链表形式串联起来，因此需要判断当前的 hook 是否是链表的第一个
  if (workInProgressHook === null) {
    // 如果当前组件的 Hook 链表为空，那么就将刚刚新建的 Hook 作为 Hook 链表的第一个节点（头结点） 
    // This is the first hook in the list
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // 如果当前组件的 Hook 链表不为空，那么就将刚刚新建的 Hook 添加到 Hook 链表的末尾（作为尾结点）
    // Append to the end of the list
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```

假如现在我们有如下的一个组件：

```js
function App() {
  const [number, setNumber] = React.useState(0); // 第一个hook
  const [num, setNum] = React.useState(1); // 第二个hook
  const dom = React.useRef(null); // 第三个hook
  React.useEffect(() => {
    // 第四个hook
    console.log(dom.current);
  }, []);
  
  return (
    <div ref={dom}>
      <div onClick={() => setNumber(number + 1)}> {number} </div>
      <div onClick={() => setNum(num + 1)}> {num}</div>
    </div>
  );
}
```

当上面的函数组件第一次初始化之后，就会形成一个 hook 链表：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-03-031041.png)
接下来我们来看更新，更新的时候会执行 `HooksDispatcherOnUpdate.updateXXX()` 对应的方法，相关代码如下：

```js hl:29-36
function updateWorkInProgressHook() {
  let nextCurrentHook;
  if (currentHook === null) {
    // 从 alternate 上获取到 fiber 对象
    const current = currentlyRenderingFiber.alternate;
    if (current !== null) {
      // 拿到第一个 hook 对象
      nextCurrentHook = current.memoizedState;
    } else {
      nextCurrentHook = null;
    }
  } else {
    // 拿到下一个 hook
    nextCurrentHook = currentHook.next;
  }

  // 更新 workInProgressHook 的指向
  // 让 workInProgressHook 指向最新的 hook
  let nextWorkInProgressHook; // 下一个要进行工作的 hook
  if (workInProgressHook === null) {
    // 当前是第一个，直接从 fiber 上获取第一个 hook
    nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
  } else {
    // 取链表的下一个 hook
    nextWorkInProgressHook = workInProgressHook.next;
  }

  // nextWorkInProgressHook 指向的是当前要工作的 hook
  if (nextWorkInProgressHook !== null) {
    // There's already a work-in-progress. Reuse it.
    // 进行复用
    workInProgressHook = nextWorkInProgressHook; 
    nextWorkInProgressHook = workInProgressHook.next;

    currentHook = nextCurrentHook;
  } else {
    // Clone from the current hook.
    // 进行克隆
    if (nextCurrentHook === null) {
      const currentFiber = currentlyRenderingFiber.alternate;
      if (currentFiber === null) {
        // This is the initial render. This branch is reached when the component
        // suspends, resumes, then renders an additional hook.
        const newHook = {
          memoizedState: null,

          baseState: null,
          baseQueue: null,
          queue: null,

          next: null,
        };
        nextCurrentHook = newHook;
      } else {
        // This is an update. We should always have a current hook.
        throw new Error("Rendered more hooks than during the previous render.");
      }
    }

    currentHook = nextCurrentHook;

    const newHook = {
      memoizedState: currentHook.memoizedState,

      baseState: currentHook.baseState,
      baseQueue: currentHook.baseQueue,
      queue: currentHook.queue,

      next: null,
    };
    // 之后的操作和 mount 时候一样
    if (workInProgressHook === null) {
      // This is the first hook in the list.
      currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
    } else {
      // Append to the end of the list.
      workInProgressHook = workInProgressHook.next = newHook;
    }
  }
  return workInProgressHook;
}
```

上面的源码中有一个非常关键的信息：

```js
// ...
if (nextWorkInProgressHook !== null) {
    // There's already a work-in-progress. Reuse it.
    // 进行复用
    workInProgressHook = nextWorkInProgressHook; 
    nextWorkInProgressHook = workInProgressHook.next;
    
    currentHook = nextCurrentHook;
}
// ...
```

==如果 `nextWorkInProgressHook` 不为 `null`，那么就复用之前的 hook，这里其实也解释了为什么 hook 不能放在条件或者循环语句中。==
==因为更新的过程中，如果通过 if 条件增加或者删除了 hook，那么在复用的时候，就会产生当前hook 的顺序和之前 hook 的顺序不一致的问题。==

例如我们把上面的代码进行修改：

```js
function App({ showNumber }) {
  let number, setNumber
  showNumber && ([ number,setNumber ] = React.useState(0)) // 第一个hooks
  const [num, setNum] = React.useState(1); // 第二个hook
  const dom = React.useRef(null); // 第三个hook
  React.useEffect(() => {
    // 第四个hook
    console.log(dom.current);
  }, []);
  
  return (
    <div ref={dom}>
      <div onClick={() => setNumber(number + 1)}> {number} </div>
      <div onClick={() => setNum(num + 1)}> {num}</div>
    </div>
  );
}
```

假如第一次父组件传递来的 `showNumber` 为 true，此时就会渲染第一个 hook，第二次渲染的时候，假如父组件传递来的 `showNumber` 为 false，那么第一个 hook 就不会执行，那么逻辑就会变成如下所示：

| *hook* 链表顺序 | 第一次     | 第二次     |
| :-------------- | :--------- | :--------- |
| 第一个 *hook*   | *useState* | *useState* |
| 第二个 *hook*   | *useState* | *useRef*   |

此时在进行复用的时候就会报错：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-03-072611.png)
第二次复用的时候发现 hook 的类型不同，`useState !==useRef` 那么就会报错。因此开发的时候一定要注意 hook 顺序的一致性。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-03-03-031320.jpg)