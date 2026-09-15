---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/00 Vue3 相比 Vue2 存在的变化/","dg-note-properties":{}}
---

关于这个话题，官方也给出了一份 Vue2 迁移到 Vue3 的指南，[详见](https://v3-migration.vuejs.org/zh/)。本篇文章则是对文档中没有提到的内容进行一个补充。

## 🔢 源码的优化

1、仓库模式

Vue2 的源码都存放在 src 目录下，然后依据功能拆分出来：

- compiler：编译器
- core：和平台无关的通用运行时代码
- platforms：平台专有代码
- shared：共享工具库代码

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1739178284833-6ffa203d-91af-4cad-9518-f88ec0c0a263.png)

详见：[https://github.com/vuejs/vue/tree/main/src](https://github.com/vuejs/vue/tree/main/src)

但是各个模块「无法独立」出来下载安装，也无法针对单个模块进行发布。

Vue3 源代码的工程改为了 Monorepo（一个仓库里面存放多个包）的形式，将模块拆分到了不同的包里面，每个包有各自的 API、类型定义以及测试，这样一来颗粒度更加细化，责任划分更加明确。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1739178538801-8ebdcddc-493c-413d-8f6f-01b3f15c8889.png)

例如只安装 reactive 包，就可以只引入 reactive 包体积会小很多。

2、类型语言

Vue2 采用的是 Flow 类型语言，Vue3 使用的是 TypeScript 类型语言，代码可读性更强。

## 🔢 性能优化

1、源码体积更小

- 移除了冷门的 API（例如`filter`、`inline-template`）；
- 生产环境使用 Rollup 进行构建，利用 Tree Shaking 减少用户代码打包的体积；

2、数据劫持的优化

Vue2 时期很多 API 还不算成熟，不得不做一些兼容性的代码，做一些妥协。

- Vue2 使用`Object.defineProperty()`;
- Vue3 使用`Proxy`;

3、编译优化

Vue 模版本质上是渲染函数的语法糖，Vue 的编译器最终会将模版编译为渲染函数。在编译器方面，Vue3 也进行了优化：

- 静态提升：提前将不变的静态内容提取出来，减少不必要的 DOM 操作；
- 预字符串化：通过预先将模板字符串化，减少渲染时的计算量；
- 缓存事件处理函数：避免每次渲染时重新创建事件处理函数，提高性能；
- Block Tree：通过将模板分解成小块，精确地管理每个部分的更新，减少不必要的 DOM 更新；
- PatchFlag：通过标记和追踪模板中发生变化的部分，优化 DOM 更新，只更新真正变化的内容，避免不必要的重渲染；

4、Diff 算法的优化

- Vue2 使用双端 Diff 算法；
- Vue3 使用快速 Diff 算法；

## 🔢 语法 API 优化

1、优化逻辑组织

- Vue2 使用 Options API，开发者需要将对应的内容都书写到`data`、`methods`、`computed`等指定位置；
- Vue3 新增 Composition API，这也是 Vue3 目前推荐的方式。这种方式的优点是代码之间的逻辑更加的紧密，不再分散，[详见](https://cn.vuejs.org/guide/extras/composition-api-faq.html)；
    - ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1739180904122-a04b75bc-6ccf-473b-ac0b-08a8f970889f.png)

2、优化逻辑复用

- Vue2 复用逻辑使用 Mixin，但是 Mixin 本身存在一些缺点：
    - 不清晰的数据来源；
    - 命名易冲突；
    - 隐式的跨 Mixin 交流；
- Vue3 复用逻辑推荐使用自定义的 Hooks；

3、创建应用实例的方式

- Vue2 使用实例化构造函数的形式；

```js
import App from './App.vue'
// 通过实例化 Vue 来创建应用
new Vue({
  el: '#app',
  components: { App },
  template: '<App/>'
})
// or
new Vue({
  render: h => h(App),
}).$mount('#app')
```

这种方式存在一个问题，那就是如果一个页面存在多个 Vue 应用（这种情况比较少见），部分配置会影响全部的 Vue 应用。

```js
Vue.use(...); // 此代码会影响所有的vue应用
Vue.mixin(...); // 此代码会影响所有的vue应用
Vue.component(...); // 此代码会影响所有的vue应用

new Vue({
  // 配置
}).$mount("#app1")

new Vue({
  // 配置
}).$mount("#app2")
```

- Vue3 使用 API 调用的方式；

```js
import { createApp } from 'vue';
import App from './App.vue'

createApp(App).mount('#app');
```

这种方式可以很多的避免上面的问题：

```js
createApp(根组件).use(...).mixin(...).component(...).mount("#app1");
createApp(根组件).mount("#app2");
```

## 🔢 引入 RFC

RFC 的全称是 Request For Comments，这是一种软件开发和开源项目中常见的「提案流程」，用于收集社区对某个新功能、改动和标准的意见和建议。

RFC 是一种文档格式，它详细描述了某个特性或更改的提议，讨论其动机、设计选择、实现细节以及潜在的影响。在通过讨论和反馈达成共识后，RFC 会被采纳或拒绝。

一份 RFC 主要的组成部分有：

1. 标题：简短描述提案的目的；
2. 摘要：简要说明提案的内容和动机；
3. 动机：解释为什么需要这个提案，解决了什么问题；
4. 详细设计：深入描述提案的设计和实现细节；
5. 潜在问题和替代方案：讨论可能存在的问题和可以考虑的替代方案；
6. 不兼容的变更：描述提案是否会引入不兼容的变更，以及这些变更的影响；

通过 RFC，Vue 的核心团队可以更好的倾听用户的需求和建议，从而开发出更加符合社区期待的功能和新特性。
