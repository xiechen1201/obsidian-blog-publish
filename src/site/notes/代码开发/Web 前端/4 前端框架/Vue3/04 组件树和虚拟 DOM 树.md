---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/04 组件树和虚拟 DOM 树/","dg-note-properties":{}}
---

在早期的前端开发时能接触到的树只有 DOM 树：

```html
<div>
  <h1>你喜欢的水果</h1>
  <ul>
    <li>西瓜</li>
    <li>香蕉</li>
    <li>苹果</li>
  </ul>
</div>
```

上面这段 HTML 的结构就会形成 DOM 树的结构：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1726017585492-19e7ea00-631b-425f-aea5-510e5268930a.png)

那什么又是组件树呢？

需要知道什么是组件？组件的出现是为了解决什么问题？

组件的本质上就是对一组 DOM 进行复用（先只关注结构，不关注行为和样式）。

可以把上面的 DOM 结构封装为一个组件，例如封装为`<Fruits />`组件。该组件就可以被用到其他的组件里面，组件和组件之间就形成了树结构，这就是组件树。

每个组件的背后都对应一组虚拟 DOM 树，虚拟 DOM 的背后又是真实 DOM 的映射：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1726017936079-ac44f5ec-797a-4b91-a5f3-dd62923a912a.png)

接下来再说什么是组件树？

组件树就是一个一个组件所形成的树结构。

什么是虚拟 DOM 树呢？

虚拟 DOM 树指的是某一个组件内部的虚拟 DOM 树结构，并非整个应用的虚拟 DOM 树结构。

和 React 不同，在 React 中虚拟 DOM 树结构是指整个应用。

<br/>

理清楚上面的概念后，有助于理解 Vue 为什么既有响应式，又有虚拟 DOM 和 diff 算法。

回顾 Vue1.x 和 Vue2.x 的响应式：

- `Object.defineProperty()`：getter（依赖收集）和 setter（通知 Watcher 进行更新）;
- Dep：相当于观察者模式中的发布者;
- Watcher：相当于观察者模式中的订阅者;

当响应式数据发生变化后，发布者会通知所有的 Watcher 重新执行对应的函数。

> [!tip]
>
> 响应式是 Vue1.x 和 Vue2.x 的核心特性，Vue3.x 依然保留了响应式，但是 Vue3.x 的响应式和 Vue1.x 、Vue2.x 不同。


但是在 Vue1.x 的时候没有虚拟 DOM，模版中每次引入一个响应式数据，就会生成一个 Watcher：

```vue
<template>
  <div class="wrapper">
    <!-- 模版中每引用一次响应式数据，就会产生一个 watcher -->
    <!-- watcher1 -->
    <div class="msg1">{{ msg }}</div>
    <!-- watcher2 -->
    <div class="msg2">{{ msg }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 和 dep 一一对应，和 watcher 一对多
      msg: 'Hello Vue 1.0'
    };
  }
};
</script>
```

这么做的特点：

- 优点：能够精确的知道哪个数据发生了变化；
- 缺点：当应用足够复杂的时候，一个应用里面会包含大量的组件，而这种设计又会导致一个组件对应多个 Watcher，这样的设计是非常消耗资源的

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1726018808850-951cc5c5-3c5e-4df8-8ec8-0ac5510f4eb5.png)

于是从 Vue2.0 版本开始就引入了虚拟 DOM，2.0 的响应式有一个非常大的变化，就是将 Watcher 的力度放大到了组件级别。也就说一个组件对应一个 Watcher，这样一来 Watcher 的数量就降低了很多。

但是这样的设计又会带来一个新的问题：1.x 的时候是可以精确的知道哪一个节点要进行更新的，但是现在因为 Watcher 是组件级别的，就只能知道是哪个组件要进行更新，组件内部具体哪一个节点要进行更新是无法得知的。

这个时候虚拟 DOM 就派上用场了，通过对 DOM 的 diff 算法，可以精确的知道哪一个节点要进行更新。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1726024656092-b54a30e8-32ec-46ef-bf30-1980dc6b0c08.png)

Vue3 的响应式在架构层面是没有变化的，仍然是响应式 + 虚拟 DOM。

- 响应式：精确到组件级别，能够知道哪一个组件更新了。不过 Vue3 的响应式是基于 Proxy 的；
- 虚拟 DOM：通过 diff 算法来计算哪一个节点需要进行更新。不过，diff 算法也不再是 Vue2 的 diff 算法，算法方面也进行了更新；
