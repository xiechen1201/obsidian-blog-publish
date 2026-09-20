---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/31 useState 和 useReducer/","dg-note-properties":{}}
---

## mount 阶段

1、useState 的 mount 阶段

```js
function mountState(initialState) {
  // 拿到 hook 对象
  const hook = mountWorkInProgressHook();
  // 如果传入的值是函数，则执行函数获取到初始值
  if (typeof initialState === "function") {
    initialState = initialState();
  }
  
  // 将初始化保存到 hook 对象的 memoizedState 和 baseState 上面
  hook.memoizedState = hook.baseState = initialState;
  const queue = {
    pending: null,
    lanes: NoLanes,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: initialState,
  };
  hook.queue = queue;
  
  // dispatch 就是用来修改状态的方法
  const dispatch = (queue.dispatch = dispatchSetState.bind(
    null,
    currentlyRenderingFiber,
    queue
  ));
  return [hook.memoizedState, dispatch];
}
```

2、useReducer 的 mount 阶段

```js
function mountReducer(reducer, initialArg, init) {
  // 创建 hook 对象
  const hook = mountWorkInProgressHook();
  let initialState;
  // 如果有 init 初始化函数，就执行该函数
  // 将执行的结果给 initialState
  if (init !== undefined) {
    initialState = init(initialArg);
  } else {
    initialState = initialArg;
  }
  
  // 将 initialState 初始值存储 hook 对象的 memoizedState 以及 baseState 上面
  hook.memoizedState = hook.baseState = initialState;
  // 创建 queue 对象
  const queue = {
    pending: null,
    lanes: NoLanes,
    dispatch: null,
    lastRenderedReducer: reducer,
    lastRenderedState: initialState,
  };
  hook.queue = queue;
  
  const dispatch = (queue.dispatch = dispatchReducerAction.bind(
    null,
    currentlyRenderingFiber,
    queue
  ));
  // 向外部返回初始值和 dispatch 修改方法
  return [hook.memoizedState, dispatch];
}
```

总结一下，mountState 和 mountReducer 的大致流程是一样的，但是有一个区别，mountState 的 queue 里面的 lastRenderedReducer 对应的是 basicStateReducer，而 mountReducer 的 queue 里面的 lastRenderedReducer 对应的是 开发者自己写的 reducer。
这里说明一个问题，useState 本质是就是 useReducer 的一个简化版，只不过在 useState 内部，会有一个内置的 reducer。

basicStateReducer 对应代码如下：

```js
function basicStateReducer(state, action) {
  return typeof action === "function" ? action(state) : action;
}
```

## update 阶段

useState 的 update 阶段：

```js
function updateState(initialState) {
  return updateReducer(basicStateReducer, initialState);
}
```

useReducer 的 update 阶段：

```js
function updateReducer(reducer, initialArg, init){
	// 获取对应的 hook
  const hook = updateWorkInProgressHook();
  // 拿到对应的更新队列
  const queue = hook.queue;
  
  queue.lastRenderedReducer = reducer;
  
  // 省略根据 update 链表计算新的 state 的逻辑
  // 这里有一套完整的关于 update 的计算流程
  
  const dispatch = queue.dispatch;
  
  return [hook.memoizedState, dispatch];
}
```