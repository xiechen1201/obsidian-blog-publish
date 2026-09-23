---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/33 【组合式 API】生命周期/","dg-note-properties":{}}
---

本篇我们将把 Vue2 和 Vue3 的生命周期函数进行对比，以下是 Vue2 整个生命周期的执行过程：

![画板](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1688451493482-2734d7ba-4b3c-4887-9097-6dce5a0d53f4.jpeg)

Vue3 生命周期执行顺序如下：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685697019486-64fb9199-511c-437c-9345-18fe3f092d38.png)

和 Vue2 流程大致都是一样的，只不过`setup()`中不能再使用`beforeCreate()`和`created()`钩子函数，因为`setup()`要先比它们两个执行！！！

以下是 Vue2 和 Vue3 声明周期函数的区别：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685697531686-8f220856-6463-4057-bb29-d955f624b1fc.png)

在 Vue3 的组合式 API 中，生命周期函数都是通过 ESModule 的形式来导入，在导入生命周期函数的时候都要在前面加上`on`且小驼峰的形式，这和选项 API 不一样：

```js
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated
} from "vue";

export default {
  setup() {
    onBeforeMount(() => console.log("onBeforeMount"));

    onMounted(() => console.log("onMounted"));

    onBeforeUpdate(() => console.log("onBeforeUpdate"));

    onUpdated(() => console.log("onUpdated"));
  }
}
```

生命周期函数都传递一个回调函数，在回调函数内部书写我们的逻辑。

生命周期函数只能同步的在`setup()`函数内部执行，因为它们都依赖内部的全局状态来定位到当前的组件实例上面。

> [!warning]
>
> 这并不意味着生命周期必须放在`setup()`中。函数也可以在一个外部函数中调用，只要调用栈是同步的，且最终起源自`setup()`就可以。


Vue3 还提供了一些用于调试处理的钩子函数：

1、`onErrorCaptured()`该函数会在子孙组件内部发生错误的时候被执行，接收 3 个参数：错误对象、触发该错误的组件实例，以及一个说明错误来源类型的信息字符串。

详见：

[组合式 API：生命周期钩子 | Vue.js](https://cn.vuejs.org/api/composition-api-lifecycle.html#onerrorcaptured)

```js
onErrorCaptured((error, instance, type) => {
  console.log(error, instance, type);
});
```

2、`onRenderTracked()`该函数会在当组件渲染过程中追踪到响应式依赖时调用，只能在开发模式下使用。

详见：

[组合式 API：生命周期钩子 | Vue.js](https://cn.vuejs.org/api/composition-api-lifecycle.html#onrendertracked)

```js
// 当组件渲染的时候会执行，可以进行 debugger 调试
onRenderTracked((e) => {
  debugger
});
```

3、`onRenderTriggered()`该函数会在组件响应式依赖的变更触发了组件渲染时调用。

详见：

[组合式 API：生命周期钩子 | Vue.js](https://cn.vuejs.org/api/composition-api-lifecycle.html#onrendertracked)

```js
// 和 onRenderTracked 一样，但是是在重新渲染的时候执行
onRenderTriggered((e) => {
  debugger
});
```
