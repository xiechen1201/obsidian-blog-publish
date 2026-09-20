---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/29 React 中的事件/","dg-note-properties":{}}
---

在 React 中有一套自己的事件系统，如果说 React 中的 FiberTree 这个数据结构是用来描述 UI 的，那么 React 里面的事件系统就是用来描述 FiberTree 和 UI 之间的交互的。

对于 ReactDOM 宿主环境，这套事件系统由两个部分：
- 合成事件对象
SyntheticEvent （合成事件对象）这个是对浏览器原生事件的一层封装，兼容了主流的浏览器，同时拥有和浏览器原生事件相同的 API，例如 stopPropagation 和 eventDefault。SyntheticEvent 存在的目的就是为了消除浏览器在事件对象上的差异。
- 模拟实现事件传播机制
利用事件委托的原理，React 会基于 FiberTree 来实现事件的捕获、目标以及冒泡的过程（类似于原生 DOM 的事件传递过程），并且在自己实现的这一套传播机制中还加入了很多新特性，例如：
- 不同的事件对应不同的优先级；
- 定制事件名称；
	- 例如在 React 中统一采用 onXXX 的驼峰写法来绑定事件
- 定制事件的行为
	- 例如 onChange 的默认行为和原生的 oninput 是相同的；

React 事件系统还考虑了很多边界的情况，因此代码量非常大，我们通过书写一个 mini 的事件系统来学习 React 事件系统的原理。

例如我们有如下的一段 JSX：

```jsx
const jsx = (
  <div onClick={(e) => console.log("click div")}>
    <h3>你好</h3>
    <button
      onClick={(e) => {
        // e.stopPropagation();
        console.log("click button");
      }}
    >
      点击
    </button>
  </div>
);
```

在上面的代码中，我们为外层的 div 以及内部的 button 都绑定了点击事件，默认情况下，点击 button 会打印出“click button”、“click div”，如果打开 `e.stopPropagation()`，那么就会阻止事件冒泡，只打印出“click button”。

可以看出，React 内部的事件系统实现了“模拟实现事件传播机制”。

## 实现 SyntheticEvent

SyntheticEvent 指的是合成事件对象，在 React 中 SyntheticEvent 会包含很多的属性和方法，我们这里只实现一个阻止冒泡：

```js
/**
 * 合成事件对象类
 */
class SyntheticEvent {
  constructor(e) {
    // 保存原生的事件对象
    this.nativeEvent = e;
  }
  // 合成事件对象需要提供一个和原生 DOM 同名的阻止冒泡的方法
  stopPropagation() {
    // 当开发者调用 stopPropagation 方法，将该合成事件对象的 _stopPropagation 设置为 true
    this._stopPropagation = true;
    if (this.nativeEvent.stopPropagation) {
      // 调用原生事件对象的 stopPropagation 方法来阻止冒泡
      this.nativeEvent.stopPropagation();
    }
  }
}
```

上面的代码中，我们创建了一个 `SyntheticEvent` 类，这个类可以用来创建合成事件对象。内部保存了原生的事件对象，还提供了一个和原生 DOM 事件同名的阻止冒泡的方法。

## 实现事件的传播机制

对于可以冒泡的事件，整个事件的传播机制实现步骤如下：
- 在根元素绑定“事件类型对应的事件回调”，所有子孙元素触发该类事件时最终会委托给根元素的事件回调函数来进行处理；
- 寻找触发事件的 DOM 元素，找到对应的 FiberNode；
- 反向遍历并执行一遍收集的所有回调函数（模拟捕获阶段的实现）；
- 正向遍历并执行一遍收集的所有回调函数（模拟冒泡阶段的实现）；

首先我们通过 `addEcent()` 来给根元素绑定事件，目的是为了实现委托：

```js hl:9
/**
 * 该方法用于给根元素绑定事件
 * @param {*} container 根元素
 * @param {*} eventType 事件类型
 */
export const addEvent = (container, eventType) => {
  container.addEventListener(eventType, (event) => {
    // 进行事件的派发
    dispatchEvent(event, eventType.toUpperCase());
  });
};
```

接下来在入口中调用 `addEvent()` 函数：

```jsx file:main.jsx hl:21
// ...
import { addEvent } from "./my-event";

const jsx = (
  <div bindCLICK={(e) => console.log("click div")}>
    <h3>你好</h3>
    <button
      bindCLICK={(e) => {
        // e.stopPropagation();
        console.log("click button");
      }}
    >
      点击
    </button>
  </div>
);

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(jsx);
// 进行根元素的事件绑定，换句话说，就是使用我们自己的事件系统
addEvent(document.getElementById("root"), "click");
```

然后回到 `addEvent()` 函数，内部使用 `dispatchEvent()` 对事件进行派发：

```js hl:21,23,28
/**
 *
 * @param {*} e 原生的事件对象
 * @param {*} eventType 事件类型，已经全部转为了大写，比如这里传递过来的是 CLICK
 */
const dispatchEvent = (event, eventType) => {
  // 实例化一个合成事件对象
  const se = new SyntheticEvent(event);
  // 拿到触发事件的元素
  const ele = event.target;
  let fiber;

  // 通过 DOM 元素找到对应的 FiberNode
  for (let prop in ele) {
    if (prop.toLocaleLowerCase().includes("fiber")) {
      fiber = ele[prop];
    }
  }

  // 找到对应的 fiberNode 之后，接下来我们需要收集路径中该事件类型所对应的所有的回调函数
  const paths = collectPaths(eventType, fiber);
  // 模拟捕获的实现
  triggerEventFlow(paths, eventType + "CAPTURE", se);
  // 模拟冒泡的实现
  // 首先需要判断是否阻止了冒泡
  // 如果没有，那么我们只需要将 paths 进行反向再遍历执行一次即可
  if (!se._stopPropagation) {
    triggerEventFlow(paths.reverse(), eventType, se);
  }
};
```

`dispatchEvent()` 函数对应的步骤：
- 实例化一个合成对象；
- 找到对应的 FiberNode；
- 从 FiberNode 一直向上找到所有该事件类型的回调函数；
- 模拟捕获的实现；
- 模拟冒泡的实现；

## 收集路径中对应的事件处理函数

```js hl:21-27
/**
 * 该方法用于收集路径中所有 eventType 类型的事件回调函数
 * @param {*} eventType 事件类型
 * @param {*} begin FiberNode
 * @returns
 * [{
 *  CLICK : function(){...}
 * },{
 *  CLICK : function(){...}
 * }]
 */
const collectPaths = (eventType, begin) => {
  const paths = []; // 存放收集到所有的事件回调函数
  // 如果不是 HostRootFiber，就一直往上遍历
  while (begin.tag !== 3) {
    const { memoizedProps, tag } = begin;
    // 如果 tag 对应的值为 5，说明是 DOM 元素对应的 FiberNode
    if (tag === 5) {
      const eventName = "bind" + eventType; // bindCLICK
      // 接下来我们来看当前的节点是否有绑定事件
      if (memoizedProps && Object.keys(memoizedProps).includes(eventName)) {
        // 如果进入该 if，说明当前这个节点绑定了对应类型的事件
        // 需要进行收集，收集到 paths 数组里面
        const pathNode = {};
        pathNode[type] = memoizedProps[eventName];
        paths.push(pathNode);
      }
      begin = begin.return;
    }
  }
  return paths;
};
```

`collectPaths()` 方法的实现思路就是从当前 FiberNode 一直向上查找，直到 HostRootFiber，收集遍历过程中 `FiberNode.memoizedProps` 属性所保存的对应的事件处理函数。

最终返回的 paths 数组保存到饿结构大致如下：

```js
[
  {
   CLICK : function(){...}
  },
  {
   CLICK : function(){...}
  }
]
```

## 捕获和冒泡的实现

由于我们是从目标元素的 FiberNode 向上遍历的，因此收集到的顺序：

```md
[
	目标元素的事件回调，
	某个祖先元素的事件回调，
	某个更上层的祖先元素的事件回调 
]
```

所以要模拟捕获阶段的实现，就需要从后往前进行遍历执行：

```js hl:17
/**
 *
 * @param {*} paths 收集到的事件回调函数的数组
 * @param {*} eventType 事件类型
 * @param {*} se 合成事件对象
 */
const triggerEventFlow = (paths, eventType, se) => {
  // 挨着挨着遍历这个数组，执行回调函数即可
  // 模拟捕获阶段的实现，所以需要从后往前遍历数组并执行回调
  for (let i = paths.length; i--; ) {
    const pathNode = paths[i];
    const callback = pathNode[eventType];
    if (callback) {
      // 存在回调函数，执行该回调
      callback.call(null, se);
    }
    if (se._stopPropagation) {
      // 说明在当前的事件回调函数中，开发者阻止继续往上冒泡
      break;
    }
  }
};
```

在执行事件回调的时候，每一次执行都需要检查一下 `_stopPropagation` 属性，如果为 true 说明当前的事件回调函数中阻止了冒泡，因此就需要停止后续的遍历。

如果是模拟冒泡阶段，只需要将 `paths` 进行反转在遍历一次并执行即可：

```js
if (!se._stopPropagation) {
	triggerEventFlow(paths.reverse(), eventType, se);
}
```

到此就实现了一个 mini 的 React 事件系统。