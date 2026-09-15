---
{"dg-publish":true,"permalink":"//web/4/vue3//40-api/","dg-note-properties":{}}
---

什么是侦听器（监听器）？

侦听器主要用于侦听依赖的数据（也就是响应式数据），当依赖发生变更的时候要能侦听到（监听到）这个值的变化，从而给开发者一个 API 接口能完成接下来自定义的任务。

```js
function test(a, b, callback) {
  const result = a + b;
  // 完成任务
  callback(result);
}

test(1, 2, (result)=>{
  console.log(result);
})
```

以上代码在运行完`a + b`的结果后就会执行`callback`，然后开发者就可以在`callback()`内编写自定义的任务，而这个`callback`就是一个函数的 API 接口。

## 🔢 基本使用

Vue 在组合式 API 同提供了一个`watch()`函数让我们可以侦听依赖的变化。

`watch()`函数的第一个参数表示要侦听的数据源，第二个参数表示当依赖数据发生变化时被执行的回调函数，在该函数内会接收两个参数分别是：依赖变化的新值、依赖变化的旧值。

简单说，就是当依赖的数据发生了变化，回调函数会立即执行。

```js
import { ref, watch } from "vue";
export default {
  setup(){
    const x = ref("test content");

    watch(x, (newVal, oldVal) => {
      console.log(`x is ${newVal}`)
    })
  }
}
```

`watch()`的第一个参数可以是不同形式的“数据源”：它可以是一个 ref (包括计算属性)、一个响应式对象、一个 getter 函数、或多个数据源组成的数组：

```js
const x = ref(0)
const y = ref(0)

// 单个 ref
watch(x, (newX) => {
  console.log(`x is ${newX}`)
})

// getter 函数
watch(
  () => x.value + y.value,
  (sum) => {
    console.log(`sum of x + y is: ${sum}`)
  }
)

// 多个来源组成的数组
watch([x, () => y.value], ([newX, newY]) => {
  console.log(`x is ${newX} and y is ${newY}`)
})
```

如果你侦听的数据源是一个 getter 函数，那么当 getter 函数内任意一个值发生变化的时候，`watch()`都会重新执行一次该函数，然后把函数返回的结果传递给 callback：

```js
watch(
  () => `我演讲的题目是「${ title.value }」，我要讲的内容是「${ content.value }」`,
  (newVal){
    console.log(newVal); // 我演讲的题目是 xxx ，我要讲的内容是 xxx
  }
)
```

但是，如果 getter 函数内两个（或多个）值同时发生了变化，`watch()`的回调只会执行一次！！！

这是因为当一次性操作两个数据变化的时候，`watch()`会进行依赖收集，缓存回调的执行，然后合并为一次来操作，这和`computed()`依赖收集很类似。

你不能直接把一个 Reactive 对象的属性传递给`watch()`进行侦听，因为这样被侦听的数据不是响应式的，这和 Reactive 对象的属性不能解构和当作函数参数进行传递是一样的道理。

```js
const obj = reactive({ count: 0 })

// 错误，因为 watch() 得到的参数是一个 number
watch(obj.count, (count) => {
  console.log(`count is: ${count}`)
})
```

你可以改写为一个 getter 函数：

```js
const obj = reactive({ count: 0 })

watch(() => obj.count, (count) => {
  console.log(`count is: ${count}`)
})
```

当内部数据发生变化的时候，整个 getter 函数将作为侦听的依赖，把函数作为一个引用。所以被侦听的数据必须是引用数据类型的对象，这样才能获取到数据的变化！！！

## 🔢 深度侦听

如果你侦听的数据源是一个响应式对象，会隐式地创建一个深层侦听器，该回调函数在所有属性的变更时都会被触发：

```js
const obj = reactive({ count: 0 });

setTimeout(() => {
  obj.count++;
}, 1000);

watch(obj, (newObj, oldObj) => {
  // 注意：`newObj` 此处和 `oldObj` 是相等的
  // 因为它们是同一个对象！
  console.log(`count is: ${newObj.count}`);
});
```

如果侦听的是一个 getter 函数返回对象的时候，那么当对象内属性发生变化的时候是侦听不到的，你需要深度侦听：

```js
const obj = reactive({ count: 0 });

setTimeout(() => {
  obj.count++;
}, 1000);

watch(
  () => obj, // count 变化无法被侦听
  (newObj, oldObj) => {
    console.log(`count is: ${newObj.count}`);
  }
);
```

我们需要给`watch()`传入第 3 个参数，该参数表示一个配置项：

```js
const obj = reactive({ count: 0 });

setTimeout(() => {
  obj.count++;
}, 1000);

watch(
  () => obj,
  (newObj, oldObj) => {
    console.log(`count is: ${newObj.count}`);
  },
  {
    // 表示深度监听
    deep: true
  }
);
```

这样不仅可以侦听到`obj`本身还可以侦听到`obj`下面属性的变化。

<br/>warning
⚠️ 注意

深度侦听需要遍历被侦听对象中的所有嵌套的属性，当用于大型数据结构时，开销很大。因此请只在必要时才使用它，并且要留意性能。

<br/>

## 🔢 即时回调的侦听器

`watch()`默认是懒加载的，仅当数据源变化时，才会执行回调。但在某些场景中，我们希望在创建侦听器时，立即执行一遍回调。

举例来说，我们想请求一些初始数据，然后在相关状态更改时重新请求数据。这个时候我们可以为`watch()`配置`immediate: true`来解决这个问题：

```js
watch(source, (newValue, oldValue) => {
  // 立即执行，且当 `source` 改变时再次执行
}, { immediate: true })
```

这样，组件会在创建后立即执行一次`watch()`的回调。

## 🔢 回调触发的时机

默认情况下，`watch()`的回调会在组件更新之前被调用，这就导致我们有时候想获取 DOM 的内容获取到的是更新之前的内容。

```html
<div ref="titleRef">{{ title }}</div>
```

```js
const title = ref("This is title.");
const titleRef = ref(null);

setTimeout(() => {
  title.value = "这是标题";
}, 1000);

watch(title, (newVal, oldVal) => {
  // 当 title 数据变化时候
  // 获取 DOM 的内容依然是 This is title.
  console.log(titleRef.value.innerText);
});
```

如果你想获取到组件更新之后的 DOM 内容，可以给`watch()`配置一个`flush: "post"`来表示在组件更新之后再执行回调：

```js
const title = ref("This is title.");
const titleRef = ref(null);

setTimeout(() => {
  title.value = "这是标题";
}, 1000);

watch(
  title,
  (newVal, oldVal) => {
    // 这样获取到的 DOM 内容就是“这是标题”
    console.log(titleRef.value.innerText);
  },
  {
    flush: "post"
  }
);
```

## 🔢 onTrack() 与 onTrigger()

`onTrack()`和`onTrigger()`是提供给开发者在开发模式下调试使用的，`onTrack()`会在当依赖源被追踪调用的时候执行，`onTrigger()`会在依赖源被更改的时候执行。

详见文档：

[深入响应式系统 | Vue.js](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#watcher-debugging)

```js
watch(
  title,
  (newVal, oldVal) => {
    console.log(newVal, oldVal);
  },
  {
    onTrack: (e) => {
      debugger;
      console.log(e);
    },
    onTrigger: (e) => {
      debugger;
      console.log(e);
    }
  }
);
```

## 🔢 停止侦听

在组合式 API 中用同步语句创建的侦听器，会自动绑定到宿主组件实例上，并且会在宿主组件卸载时自动停止。因此，在大多数情况下，你无需关心怎么停止一个侦听器。

需要注意的是，侦听器必须用同步语句创建：如果用异步回调创建一个侦听器，那么它不会绑定到当前组件上，你必须手动停止它，以防内存泄漏。

```js
// 它会自动停止
watchEffect(() => {})

// ...这个则不会！
setTimeout(() => {
  watchEffect(() => {})
}, 100)
```

要手动停止一个侦听器，请调用`watch()`或`watchEffect()`返回的函数：

```js
const unwatch = watchEffect(() => {})

// ...当该侦听器不再需要时
unwatch()
```

## 🔢 watchEffect()

Vue 还提供了`watchEffect()`方法用来监听数据：

```js
watchEffect(() => {
  // do something
});
```

`watchEffect()`的名字就是`watch`+`Effect`，`Effect`表示副作用，那么什么是前端中的副作用呢？

简单的来说，只要是跟函数外部环境发生的交互就都是副作用。从“副作用”这个词语来看，它更多的情况在于“改变系统状态”。包括：

- 更改文件系统
- 往数据库插入记录
- 发送一个 http 请求
- 可变数据
- 打印/log
- 获取用户输入
- DOM 查询
- 访问系统状态

而`watchEffect()`函数就是让我们在回调函数内处理副作用的，它会观察副作用函数内部的依赖数据是不是发生了变化，然后执行回调函数。

`watchEffect()`和`watch()`的区别：

1、`watch()`需要指定被侦听的依赖数据。`watchEffect()`不需要指定被侦听的数据，它会的追踪回调函数中的响应式依赖数据。

2、`watch()`是懒执行的，并不是在某个时间立即执行的，而是看侦听的数据是否发生了变化来决定是否执行回调（这里不指`immediate: true`的情况）。`watchEffect()`是在侦听器创建和依赖的数据发生变更的时候都会执行回调。

3、`watch()`可以在回调函数内通过形参的方式拿到依赖变化的新值和旧值。`watchEffect()`无法获取到旧值，新值也是通过依赖数据本身来获取的，无法在回调函数内通过形参的方式获取值。

综上，如果你想拿到依赖数据的新值和旧值，使用`watch()`更加合适。

`watchEffect()`是会在被创建的时候首先执行一次，这其实是有目的的，是为了收集依赖数据。

```js
watchEffect(() => {
  // 触发 title 的 getter 机制，依赖被收集
  console.log(title.value);
});
```

`watchEffect()`方法同样也支持对侦听器进行配置，`flush: "pre"`是它的默认配置，表示在组件挂载、「组件更新之前」执行副作用回调函数，是异步执行的。如果一次性改变多个依赖数据，那么回调函数「只会执行一次」：

```js
const title = ref("This is title.");
const content = ref("This is content.");

watchEffect(
  () => {
    const str = title.value + "--" + content.value;
    console.log(str);
  },
  {
    flush: "pre"
  }
);

setTimeout(() => {
  // 同时更改 title 和 content 的值
  title.value = "这是标题。";
  content.value = "这是内容。";
}, 1000);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1687144907243-b94d18d8-93e1-4021-8603-6cc6367732b7.gif)

当给`watchEffect()`配置`flush: "post"`的时候，和`watch()`效果是一样的，都会在组件挂载，「组件更新之后」执行回调函数：

```js
watchEffect(
  () => {
    const str = title.value + "--" + content.value;
    console.log(str);
  },
  {
    flush: "post"
  }
);
```

当给`watchEffect()`配置`flush: "sync"`的时候，也会在组件挂载、「组件更新之前」执行回调函数，但是不会进行缓存，它是同步执行的。简单说，就是有多少个依赖变化，那么回调函数就会执行多少次。

```js
watchEffect(
  () => {
    const str = title.value + "--" + content.value;
    console.log(str);
  },
  {
    flush: "sync"
  }
);

setTimeout(() => {
  // 同时更改 title 和 content 的值
  title.value = "这是标题。";
  content.value = "这是内容。";
}, 1000);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1687145409753-917fd98e-5556-4e9a-940f-282c7a7d5bcd.gif)

<br/>warning
⚠️ 注意

你应该谨慎使用，因为如果有多个属性同时更新，这将导致一些性能和数据一致性的问题。

<br/>

## 🔢 onCleanup

`watchEffect()`的第一个参数是一个副作用回调函数，这个我们上面已经说过了，而这个副作用的回调函数也有一个参数，该参数也是一个函数，这就是`onCleanup`。

```js
watchEffect((onCleanup) => {
  onCleanup(()=>{
    // do something
  })
})
```

`onCleanup`的作用？官方是这么说的：

> 第一个参数就是要运行的副作用函数。这个副作用函数的参数也是一个函数，用来注册清理回调。清理回调会在该副作用下一次执行前被调用，可以用来清理无效的副作用。
>

什么意思呢？我们通过一个案例来解释：

```js
function sendRequest(id) {
  let timer = null;

  function response() {
    return new Promise((resovle, reject) => {
      timer = setTimeout(() => {
        resovle(`ID为「${id}」的结果是「1234567890」`);
      }, 1000);
    });
  }

  function cancel() {
    clearTimeout(t);
  }

  return { response, cancel };
}
```

以上代码，我们模拟了一个 Ajax 的请求，暴露出去一个`response()`方法用于发起请求，`cancel()`用于取消请求。

```html
<template>
  <div class="">
    <button @click="changeID">请求数据</button>
    {{ id }}
  </div>
</template>
```

```js
const id = ref(new Date().getTime());
const changeID = () => (id.value = new Date().getTime());

watchEffect(async () => {
  console.log(1);
  const { response, cancel } = sendRequest(id.value);

  let result = await response();
  console.log(result);
});
```

以上代码，我们通过点击按钮来更改`id`，当`id`改变后`watchEffect()`就会执行回调。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1687147044356-e5703062-d2f0-4321-bb2a-7eff388aab43.gif)

通过图片我们可以看到一个现象：当我在请求数据返回后再点击下一次请求这样是没问题的。可是当我在数据请求还没返回的时候连续点击，这样就会连续触发`watchEffect()`的回调，也就是连续发起请求。

所以，我需要想一个办法就是当我点击按钮的时候，如果上一次的请求还没有返回，那么我就停止上一次的请求并发起一个新的请求。

嗯。。。好像可以用函数防抖来优化一下哈。

而这里的`onCleanup()`方法，就是为了解决这样类似的问题而产生的，整个流程类似函数防抖的流程。

`onCleanup()`方法就是为了清除上一次副作用回调函数中的某些程序（例如网络请求），`onCleanup()`会在下一次回调执行之前执行，然后再执行副作用回调函数。

<br/>

```js
const id = ref(new Date().getTime());
const changeID = () => (id.value = new Date().getTime());

watchEffect(async (onCleanup) => {
  console.log(1);
  const { response, cancel } = sendRequrst(id.value);

  // 清除上一次的请求任务
  onCleanup(cancel);

  let result = await response();
  console.log(result);
});
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1687154652961-5f593def-fc1b-404e-b8df-047a87b1f565.gif)

这样，当我们连续点击按钮的时候，请求任务都会被清除停止，然后在重新发起一个请求！
