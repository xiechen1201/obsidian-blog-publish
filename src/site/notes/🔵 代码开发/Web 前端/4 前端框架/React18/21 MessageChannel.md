---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/React18/21 MessageChannel/","dg-note-properties":{}}
---

我们在学习「事件循环」的时候可能都接触过下面的这个图：
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-29-021951.gif)
以上流程：
- 首先会执行同步代码，同步代码执行时如果遇到异步代码就会放入 webapis 里面进行执行；
- 当 webapis 执行完毕后会将结果放入到 task queue 任务队列中；
- 当同步代码执行完成后，就会从任务队列中取出一个一个的任务进行执行；

如果将「事件循环」和「浏览器渲染」组合到一起就是下面的流程：

```md
                浏览器主线程（Main Thread）

                         ↓

              执行同步 JavaScript 代码

                         ↓

        遇到异步任务（定时器、网络请求、事件等）

                         ↓

              放入 Web APIs 处理

                         ↓

        Web APIs 执行完成后产生回调函数

                         ↓

             放入任务队列 Task Queue

                         ↓

          当前同步代码执行完成（Call Stack 清空）

                         ↓

              Event Loop 检查任务队列

                         ↓

          取出任务放入调用栈 Call Stack

                         ↓

              执行任务中的 JavaScript

                         ↓

                 一轮事件循环结束


                         ↓

              浏览器判断是否需要渲染

                         ↓

                  执行 Render Pipeline

                         ↓

        Style（样式计算）
              ↓
        Layout（布局计算）
              ↓
        Paint（绘制）
              ↓
        Composite（合成）


                         ↓

              显示下一帧画面

                         ↓

             进入下一轮 Event Loop
```

从上面的流程可以看出，每一次事件循环都会从任务队列中取出一个任务来执行。

之前有提到，大多数设备的刷新率为 60hz，也就是 1s 要绘制 60 次，这就意味着浏览器每隔 16.66ms 就需要重新渲染一次。也就是说，每次事件循环会取出一个任务来执行，如果执行完成后，依旧没有到达 16.66ms（浏览器需要重渲染的时间），那么就会进行下一次循环，再次从队列中取出一个任务来执行，以此类推，直到浏览器需要重新渲染。

我们在做动画效果的时候可能接触过 requestAnimationFrame API，这个 API 会在每一次重新绘制之前执行一次，这个 API 也是专门用来做动画的 API。
该 API 出现之前，我们可能使用的都是 `setTimeout` 来进行动画的操作，使用 requestAnimationFrame API 最大的优点就是频率和浏览器重新渲染的频率一致。
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-29-023954.png)
（*黄色表示要执行的 JS 任务）

requestAnimationFrame 就不会存在这个问题，因为它是在渲染之前，保证了和浏览器渲染是同频的。
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-29-024236.png)
==这里顺便提一下微任务，如果在事件循环中微任务队列中存在微任务，那么事件循环会在循环一次的时候，把整个微任务都清空，然后才会去执行浏览器渲染，也就是说微任务会阻塞主线程的渲染。==

每一次事件循环时这几种任务的区别：
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-29-024700.gif)
## 为什么 React 选择了 MessageChannel？

`MessageChannel` 本事是用来做消息通信的，允许我们创建一个消息通道，通过它的两个 `MessagePort` 来进行消息的发送和接收。

基本使用：

```js
const channel = new MessageChannel();
// 两个信息端口，这两个信息端口可以进行信息的通信
const port1 = channel.port1;
const port2 = channel.port2;
btn1.onclick = function(){
  // 给 port1 发消息
  // 那么这个信息就应该由 port2 来进行发送
  port2.postMessage(content.value);
}
// port1 需要监听发送给自己的消息
port1.onmessage = function(event){
  console.log(`port1 收到了来自 port2 的消息：${event.data}`);
}

btn2.onclick = function(){
  // 给 port2 发消息
  // 那么这个信息就应该由 port1 来进行发送
  port1.postMessage(content.value);
}
// port2 需要监听发送给自己的消息
port2.onmessage = function(event){
  console.log(`port2 收到了来自 port1 的消息：${event.data}`);
}
```

`MessageChannel` 和 React 中的 Scheduler 有什么关系呢？
之前我们提到 Scheduler 是用来调度任务的，调度任务需要满足两个条件：
- JS 暂停，将主线程归还给浏览器，让浏览器可以有序的重新渲染页面；
- 暂停了的 JS（说明没有执行完成），需要在下一次接着来执行；
那么这里自然而然的就会使用到事件循环，我们可以将没有执行完成的 JS 放入到任务队列中，下一次事件循环的时候再次取出来执行。

那么如何将没有执行完成的任务放入到任务队列中呢？
这里就需要参数一个宏任务，因此可以使用 `MessageChannel`，因为这个 API 可以产生宏任务。

## 为什么不选择 setTimeout？

我们都知道 `setTimeout(fn, 0)` 也可以产生一个宏任务，但是 React 团队为什么没有使用呢？

这是因为 `setTimeout` 在嵌套的层级中超过 5 层，那么 timeout（延时）如果小于 4ms，那么就会自动设置为 4ms。

```cardlink
url: https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#dom-settimeout
title: "HTML Standard"
host: html.spec.whatwg.org
favicon: https://resources.whatwg.org/logo.svg
```

> If nesting level is greater than 5, and timeout is less than 4, then set timeout to 4.

例如下面的示例：

```js
let count = 0; // 计数器
let startTime = new Date(); // 获取当前的时间戳
console.log("start time:", 0, 0);

function fn(){
  setTimeout(function(){
    console.log("exec time:", ++count, new Date() - startTime);
    if(count === 50){
      return;
    }
    fn();
  },0)
}

fn();
```
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2022-12-29-031030.png)（* 部分执行结果截图）

正是因为这个原因 React 团队没有使用 `setTimeout` 来产生宏任务，因为 4ms 的浪费还是不可忽视的。

## 为什么没有选择 requestAnimationFrame？

选择 `requestAnimationFrame` 这个 API 也不合适，因为这会 API 只会在浏览器重新渲染之前才能执行一次/。如果我们包装成一个任务，放入任务队列中，那么只要没到重新渲染的时间，就可以一直从任务队列里面获取任务来执行。

同时 `requestAnimationFrame` API 还存在一定的兼容问题，safari 和 edge 浏览器是将 `requestAnimationFrame` 放到渲染之后执行的，chrome 和 firefox 是将 `requestAnimationFrame` 放到渲染之前执行的，所以这里存在不同的浏览器有不同的执行顺序的问题。

## 为什么没有包装为微任务？

这和微任务的执行机制有关系，微任务的队列会在浏览器重新渲染之前一直执行，会阻塞主线程，无法实现将主线程归还给浏览器的目的。