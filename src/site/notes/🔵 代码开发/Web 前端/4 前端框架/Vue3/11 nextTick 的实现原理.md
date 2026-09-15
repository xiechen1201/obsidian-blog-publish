---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/11 nextTick 的实现原理/","dg-note-properties":{}}
---

Vue 中的`nextTick()`函数我们会经常的使用。我们先用一段普通的代码来看下什么情况下应该使用`nextTick`:

```vue
<template>
  <div id="counter">
    <p>{{ count }}</p>
    <button @click="increment">增加计数</button>
  </div>
</template>

<script setup>
  import { ref } from 'vue'

  const count = ref(0)

  const increment = () => {
    for (let i = 1; i <= 1000; i++) {
      count.value = i
    }
  }
</script>
```

思考一下，上面的代码中点击按钮，页面会渲染几次呢？

实际上只会渲染一次，这是因为我们的`increment`代码是同步更改`count`的值。而同步代码多次对响应式数据进行更改会被合并为一次，之后会根据最终的更改结果「异步」的去更新 DOM。

如果不进行更改合并，并且同步的去更改 DOM，那么就会出现只要数据一变化，那么就会立即更新 DOM，会导致频繁的重绘和重排，这会非常的消耗性能。

如果是异步又会带来什么问题呢？

那就是无法及时的获取到更新后的 DOM。因为获取 DOM 的代码是同步的，DOM 的更新却是异步的，同步的代码会优先异步代码的执行。

```js
const increment = () => {
  for (let i = 1; i <= 1000; i++) {
    count.value = i
  }

  console.log('最新的数据：', count.value)
  console.log('通过DOM拿textContent数据：', document.getElementById('counter').textContent)
  console.log('通过DOM拿innerHTML数据：', document.getElementById('counter').innerHTML)
}
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1739257332806-47c24c6d-eb35-479a-af1a-c58409b6c75c.png)

要解决这个问题，我们只能想办法让获取最新 DOM 的代码转换为一个异步代码。例如包装成一个「微任务」，当浏览器完成渲染后，就会立即执行「微任务」：

```js
const increment = () => {
  for (let i = 1; i <= 1000; i++) {
    count.value = i
  }

  Promise.resolve().then(() => {
    console.log('最新的数据：', count.value)
    console.log('通过DOM拿textContent数据：', document.getElementById('counter').textContent)
    console.log('通过DOM拿innerHTML数据：', document.getElementById('counter').innerHTML)
  })
}
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1739257251029-6326a128-1ceb-45ac-ad96-7ac22478d23e.png)

可以看到包装成一个微任务后，我们就可以拿到最新的 DOM 内容了。

实际上`nextTick()`帮我们做的就是上面这个事情，也就是将一个任务包装为一个微任务：

```js
const increment = () => {
  for (let i = 1; i <= 1000; i++) {
    count.value = i
  }

  nextTick(() => {
    console.log('最新的数据：', count.value)
    console.log('通过DOM拿textContent数据：', document.getElementById('counter').textContent)
    console.log('通过DOM拿innerHTML数据：', document.getElementById('counter').innerHTML)
  })
}
```

`nextTick()`返回的是一个 Promise，因此我们可以使用`async/await`将上面的代码更改一下：

```js
const increment = async () => {
  for (let i = 1; i <= 1000; i++) {
    count.value = i
  }

  await nextTick()
  console.log('最新的数据：', count.value)
  console.log('通过DOM拿textContent数据：', document.getElementById('counter').textContent)
  console.log('通过DOM拿innerHTML数据：', document.getElementById('counter').innerHTML)
}
```

下面我们看下`nextTick`的源码是如何实现的（主要看`nextTick`方法）：

```ts
// 创建一个已经解析的 Promise 对象，这个 Promise 会立即被解决，
// 用于创建一个微任务（microtask）。
const resolvedPromise = /*#__PURE__*/ Promise.resolve() as Promise<any>

// 一个全局变量，用于跟踪当前的刷新 Promise。
// 初始状态为 null，表示当前没有刷新任务。
let currentFlushPromise: Promise<void> | null = null

// queueFlush 函数负责将刷新任务（flushJobs）放入微任务队列。
// 这是 Vue 的异步更新机制的核心部分，用于优化性能。
function queueFlush() {
  // 检查是否已经在刷新（isFlushing）或者刷新任务是否已被挂起（isFlushPending）。
  if (!isFlushing && !isFlushPending) {
    // 设置 isFlushPending 为 true，表示刷新任务已被挂起，正在等待执行。
    isFlushPending = true
    // 将 currentFlushPromise 设置为 resolvedPromise.then(flushJobs)
    // 这将创建一个微任务，当 resolvedPromise 被解决时，执行 flushJobs 函数。
    currentFlushPromise = resolvedPromise.then(flushJobs)
  }
}

// nextTick 函数用于在下一个 DOM 更新循环之后执行一个回调函数。
// 它返回一个 Promise，这个 Promise 会在 DOM 更新完成后解决。
export function nextTick<T = void, R = void>(
  this: T,
  fn?: (this: T) => R,  // 可选的回调函数，在 DOM 更新之后执行
): Promise<Awaited<R>> {
  // 如果 currentFlushPromise 不为 null，使用它；否则使用 resolvedPromise。
  // 这样可以确保在 DOM 更新之后再执行回调。
  const p = currentFlushPromise || resolvedPromise

  // 如果传入了回调函数 fn，返回一个新的 Promise，在 p 解决之后执行 fn。
  // 使用 this 绑定来确保回调函数的上下文正确。
  return fn ? p.then(this ? fn.bind(this) : fn) : p
  // 如果没有传入回调函数 fn，直接返回 Promise p，这样外部代码可以使用 await 等待 DOM 更新完成。
}
```

所以，总结一下`nextTick()`方法本质上就是将回调函数包装为一个微任务并放在微任务队列，这样浏览器在完成渲染任务后会优先执行微任务。

在 Vue2 中和 Vue3 还有一些不同：

- Vue2 为了浏览器的兼容性，会根据不同的环境选择不同的包装方式；
    - 优先使用 Promise，因为这个它是现代浏览器最有效的微任务实现；
    - 如果不支持 Promise，则使用 MutationObserver，这是另外一种微任务机制；
    - 在 IE 的环境下，使用的是`setImmediate()`方法（现在已经不推荐使用），这是一种接近微任务的宏任务；
    - 最后是`setTimeout(fn, 0)`进行兜底，这也是一个宏任务，但会在 下一个事件循环中尽快执行；
- Vue3 则只考虑现代浏览器环境，直接使用 Promise 来实现微任务的包装，这样做的好处在于代码更加的简洁，性能更高，因为不需要处理多种环境的兼容性问题；

最后我们使用一到代码题来进行收尾：

```vue
<template>
  <div>{{ rCount }}</div>
</template>

<script setup>
  import { ref } from 'vue';
  const count = 0;
  const rCount = ref(count);

  for(let i = 1; i <= 5; ++i){
    setTimeout(()=>{
      rCount.value = i;
    }, 0);
  }
</script>
```

以上代码会触发几次页面更新？

答案是 6 次，具体流程如下：

- 首先页面初始化的时候会被触发一次，接着就是循环立即执行，但是内部都是`setTimeout`的宏任务；
- 循环执行完成后，同步代码也就执行完成了，开始事件循环，由于目前没有微任务，所以执行一个宏任务；
- 执行`rCount.value = 1`，宏任务执行完成，继续事件循环；
- 由于更改`rCount`都是异步的（宏任务），所以无法将多次更改进行合并，接着 DOM 会被更新，更新完成后执行微任务；
- 继续下一次事件循环，执行宏任务...

我们可以使用`onUpdated`钩子函数验证一下：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1739258652044-4ffca14a-698b-4eef-98e6-0411aafe4dde.png)
