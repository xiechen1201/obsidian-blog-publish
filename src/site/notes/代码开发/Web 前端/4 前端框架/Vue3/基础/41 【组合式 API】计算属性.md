---
{"dg-publish":true,"permalink":"//web/4/vue3//41-api/","dg-note-properties":{}}
---

组合式 API 中的计算属性`computed()`和选项式 API 中的`computed`作用是一样的，都是为了抽离模版中复杂的逻辑计算，让模版看起来更简洁。

组合式 API 中的`computed()`会返回一个 ComputedRefImpl 对象：

```js
const state = reactive({
  a: 1,
  b: 2
})

const result = computed(() => state.a + state.b);

console.log(result);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1687163964534-02469470-b796-4e5d-b9a2-55abe07ae899.png)

和 Ref 对象一样，也需要通过 .value 才能访问到它的值，也会在模版中帮我们自动解包。不过，ComputedRefImpl 不再是一个普通的 Ref 对象。

`computed()`的执行时机：

1、创建`computed()`的时候执行回调；

2、回调内部依赖发生变化的时候执行回调，并把计算后的结果返回出去；

`watch()`和`computed()`的区别：

1、`watch()`侦听一个依赖的变化然后执行回调，可以拿到新值和旧值，最后交给开发者在回调函数内继完成接下来的任务。

2、`computed()`也会侦听回调内的依赖变更然后执行回调，但是它会返回一个结果，更多的是为了抽离模版中的复杂计算逻辑。所以不要再`computed()`回调内编写你的逻辑，包括：操作 DOM、发起异步请求、尝试更改`computed()`返回的值。`computed()`再设计上就是 readonly 只读的，这是因为`computed()`的值变化应该依赖回调内的数据，而不是手动的进行更改，所以你应该避免手动更改`computed()`计算的结果。

综上整体来说，`watch()`是为了完成逻辑的，`computed()`是为了完成计算值的。

当然，计算属性也可以使用一个函数来代替，但是他们之间存在很大的区别：

```js
const state = reactive({
  a: 1,
  b: 2
})

const result = () => state.a + state.b;

console.log(result());
```

当组件更新的时候，这个函数每次都会被调用执行。而`computed()`会基于响应式依赖进行缓存。

一个计算属性仅会在其响应式依赖更新时才重新计算。这意味着只要`state.a`或者`state.b`不改变，无论多少次访问`result`都会立即返回先前的计算结果，而不用重复执行 getter 函数。

相比之下，方法调用总是会在重渲染发生时再次执行函数。

<br/>
