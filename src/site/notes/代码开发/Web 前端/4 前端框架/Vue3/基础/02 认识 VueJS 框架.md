---
{"dg-publish":true,"permalink":"//web/4/vue3//02-vue-js/","dg-note-properties":{}}
---

[Vue.js - 渐进式 JavaScript 框架 | Vue.js](https://cn.vuejs.org/)

<br/>warning
⚠️ 注意

本文档都是基于`Vue3`进行输出描述的！！！

<br/>

## 🔢 渐进式框架

我们打开`Vue`官方文档的时候，首页就能看到一个醒目的大字：渐进式`JavaScript`框架。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1672371056846-e0ba9fd3-6bdb-49df-b232-71ee3c96e77b.png)

那到底什么是渐进式框架呢？

渐进式框架是`Vue`对比其他框架后，生成的特定名词，以下是前端三大框架的区别：

1、[Angular](https://angular.cn/) 是一个综合性框架，一个开放的平台，它更加关注的是项目应用，适合大型项目的开发，因为框架内部的集成非常的完整。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1672371273138-ff26e851-93c4-487a-b0fd-214e8085dffc.png)

2、[React](https://zh-hans.reactjs.org/) 是用于构建「用户界面」的`JS`库，更关注于「视图层」，重点在于如何把数据渲染到视图层。使用路由必须依赖 [react-router](https://react-guide.github.io/react-router-cn/) ，`react-router`是一个单独的库，并不在`React`中集成。

3、[Vue](https://cn.vuejs.org/) 也只关注于视图层，关于如何把数据渲染到页面，和`React`不同的是`Vue`把 [Vue Router](https://router.vuejs.org/zh/index.html)、[Vuex](https://vuex.vuejs.org/zh/)、[Nuxt 服务端渲染](https://nuxt.com/)进行选择集成（你需要使用你就安装，不使用就不必安装，且没有太大的学习难度），`Vue`处于`Angular`和`React`之间。

`Vue`和`React`都是自下而上进行开发，从最开始的视图层进行开发，先关注视图层，需要状态管理和路由你就安装集成，这就是渐进式！！！

<br/>

而`React`相对`Vue`提供的`API`相对比较少，很多需求代码都需要我们自己去实现，这就是`React`上手比较困难的原因。

## 🔢 数据绑定和数据流

`Vue`另外两个重要的概念就是「数据绑定」和「数据流」。

- 数据绑定就是数据和视图渲染之间的关系。
    - `React`是单向数据绑定，通过`event`事件触发`state`来更改视图，只能通过事件去改变视图，视图无法去更改数据。
    - `Vue`是双向数据绑定，和`Angular`类似，通过`v-model`机制来实现视图变化更改数据。
- 数据流就是数据流向的方向，简单说就是父子组件数据按照什么方向流动。
    - `React`和`Vue`都是单向数据流，父组件传递数据给子组件作为`props`，子组件不能直接去更改`props`数据，只能`emit`出一个事件，通知父组件进行数据的更改！

## 🔢 创建 Vue 应用

可以通过`cdn`、`vite`和`vue-cli`来创建`Vue`项目，详见文档：

[快速上手 | Vue.js](https://cn.vuejs.org/guide/quick-start.html)

因为本文档是`Vue`的深度学习，所以这里就不叙述`Vue`的基本使用啦，请查看`Vue`文档进行学习，文档已经写的非常清楚了。

<br/>
