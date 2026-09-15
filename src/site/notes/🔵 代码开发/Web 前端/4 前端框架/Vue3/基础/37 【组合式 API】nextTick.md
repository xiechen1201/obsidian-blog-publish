---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/基础/37 【组合式 API】nextTick/","dg-note-properties":{}}
---

无论是选项式 API 还是组合式 API 我们能可以通过 Ref 来拿到元素或组件的引用。

```vue
<template>
  <div ref="divRef">Hello Vue3!</div>
  <Child ref="childRef" />
</template>

<script>
  import { ref, onMounted } from "vue";
  import Child from "./components/Child.vue";

  export default {
    components: {
      Child
    },
    setup() {
      const divRef = ref(null);
      const childRef = ref(null);

      console.log(divRef);

      console.log(divRef.value);
      console.log(childRef.value);

      return {
        divRef,
        childRef
      };
    }
  };
</script>

<style lang="scss" scoped></style>
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686294885458-8793106b-c441-42c1-bf4a-e60cafe3c7a2.png)

<br/>warning
⚠️ 注意

我们在代码中分别打印了`divRef`和`divRef.value`，从图中可以看到我们明明拿到了`div`元素对象。

这其实是因为我们在控制台点击三角形的时候会触发 getter 机制，此时我们的程序已经跑完，你再去 getter 肯定能拿到`div`元素。

<br/>

通常情况下，我们操作一个响应式数据是很正常的操作：

```vue
<template>
  <div class="">This is Child component!</div>
  <h1 ref="h1Ref">{{ state.title }}</h1>
  <button @click="setTitle">Set title</button>
</template>

<script>
  import { ref, reactive, nextTick } from "vue";

  export default {
    setup() {
      const h1Ref = ref(null);
      const state = reactive({
        title: "This is title."
      });
      const setTitle = () => {
        state.title = "这是标题.";
      };

      return {
        h1Ref,
        state,
        setTitle
      };
    }
  };
</script>

<style lang="scss" scoped></style>
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686295189835-4765c63e-90a9-4960-b1aa-67d9c34a48ca.png)![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686295197530-43bdc91b-7bc7-42bd-9207-233ede20b10f.png)

以上代码，当你点击按钮的时候会更改`state.title`的值，相应的页面视图也会进行更新。

某些情况下，我们可能需要更改完数据后立马获取元素：

```js
const setTitle = () => {
  state.title = "这是标题.";
  console.log(h1Ref.value);
};
```

这时，你会发现获取的 DOM 引用的值还是 This is title.

这是怎么回事呢？下面是[ Vue 文档的原话](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#dom-update-timing) ：

> 当你更改响应式状态后，DOM 会自动更新。然而，你得注意 DOM 的更新并不是同步的。相反，Vue 将缓冲它们直到更新周期的 “下个时机” 以确保无论你进行了多少次状态更改，每个组件都只更新一次。
>

上面这段话的意思是：当数据状态更改后，DOM 是不会立马执行的，DOM 的更新是非同步的，Vue 会把 DOM 更新的任务缓存到一个队列中进行等待，当数据状态全部更改完成后再一次性的更新 DOM。

而如果你想拿到 DOM 更新后的内容，你可以使用 Vue 提供的`nextTick()`方法：

```js
const setTitle = () => {
  state.title = "这是标题.";

  nextTick(() => {
    console.log(h1Ref.value);
  });
};
```

`nextTick()`方法会在状态变更后立即执行，但是回调方法却是在 DOM 更新后才执行！！！

另外，`nextTick()`返回的是一个 Promise：

```js
const setTitle = () => {
  state.title = "这是标题.";

  const res = nextTick();
  console.log(res);
};
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686295855611-22d7a5f8-eb6a-4fe4-88c2-78cd32ffc254.png)

所以我们可以使用`await`进行中止等待：

```js
const setTitle = async () => {
  state.title = "这是标题.";

  /* nextTick(() => {
      console.log(h1Ref.value);
    }); */

  // 也可以 await 执行
  await nextTick();
  console.log(h1Ref.value);
};
```
