---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/21 keep-alive、动态组件、异步组件/","dg-note-properties":{}}
---

`<keep-alive>`是 Vue 内部提供的一个组件；我们在开发的时候，一般切换组件的时候，组件中的内容都会进行重新初始化，而`<keep-alive>`组件会缓存组件内的状态，目的是避免反复渲染导致的性能问题。

例如：我有一个登录的需求，登录方式包括账号密码、扫描二维码、手机号验证码三个 Tab 登录方式，我在填写账号密码的时候切换到二维码登录，再次从二维码登录切换到账户密码的登录，我们刚才输入的信息就会全部丢失，我们就可以使用`<keep-alive>`帮我们解决这个问题。

`<keep-alive>`组件通常还会和`<component>`组件进行联用，`<component>`也是 Vue 内部提供的一个组件，作用是「动态渲染」一个组件。我们在开发中，某些组件的渲染是不确定的，是需要根据交互的操作来决定渲染哪个组件，`<component>`就是帮助我们动态的渲染组件的，它接收一个`is`属性来注定组件的名字。

```vue
<template>
  <KeepAlive>
    <component :is="activeComponent" />
  </KeepAlive>
</tmplate>
```

`<component>`的`is`属性接受如下值：

- 被注册的组件名
- 导入的组件对象
- 你也可以使用`is`来创建一般的 HTML 元素。

当使用`<component :is="...">`来在多个组件间作切换时，被切换掉的组件会被卸载。我们可以通过`<keep-alive>`强制被切换掉的组件仍然保持“存活”的状态。

那什么又是异步组件呢？

我们在很多时候没有必要在当前页面中一次性的把所有的组件都导入进来，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块。

Vue2 和 Vue3 的做法存在差异，下面我们对比来看一下：

```js
// Vue2 异步加载组件

Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // 向 `resolve` 回调传递组件定义
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。Vue 只有在这个组件需要被渲染的时候才会触发该工厂函数，且会把结果缓存起来供未来重渲染。

另外一个推荐的做法是将异步组件和 Webpack 的 code-splitting 功能一起配合使用：

```js
// Vue2 异步加载组件

Vue.component('async-webpack-example', function (resolve) {
  // 这个特殊的 `require` 语法将会告诉 webpack
  // 自动将你的构建代码切割成多个包，这些包
  // 会通过 Ajax 请求加载
  require(['./my-async-component'], resolve)
})
```

或者直接通过 import 来动态加载，import() 加载后本身就是一个 Promise：

```js
Vue.component(
  'async-webpack-example',
  // 这个动态导入会返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```

另外也可以局部注册：

```js
import AccountLogin from "./AccountLogin.vue";

export default {
  components: {
    AccountLogin,
    QrcodeLogin: () => import("./QrcodeLogin.vue"),
    MobileLogin: () => import("./MobileLogin.vue")
  }
}
```

回到 Vue3 ，你需要从 Vue 中解构出`defineAsyncComponent`方法来定义异步组件：

详见：

[异步组件 | Vue.js](https://cn.vuejs.org/guide/components/async.html)

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    resolve(/* 获取到的组件 */)
  })
})
```

`defineAsyncComponent()`方法接收一个返回 Promise 的加载函数。这个 Promise 的 resolve 回调方法应该在从服务器获得组件定义时调用。

同样地，你也可以配合 import() 来动态加载：

```js
import { defineAsyncComponent } from 'vue'

export default {
  // 局部注册
  components: {
    AdminPage: defineAsyncComponent(() =>
      import('./components/AdminPageComponent.vue')
    )
  }
}
```

```js
// 全局注册
app.component('MyComponent', defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
))
```

异步操作不可避免地会涉及到加载和错误状态，因此`defineAsyncComponent()`也支持在高级选项中处理这些状态：

```js
import { defineAsyncComponent } from "vue";
import Loding from "./Loding.vue";
import Error from "./Error.vue";

// 1 秒后加载成功
export const Intro = defineAsyncComponent({
  loader: () =>
    new Promise((resolve) => {
      setTimeout(function () {
        resolve(import("./Intro.vue"));
      }, 1000);
    }),
  loadingComponent: Loding, // 加载时要显示的组件
  errorComponent: Error // 加载失败要显示的组件
});

// 2 秒后加载成功
export const Article = defineAsyncComponent(() => {
  return new Promise((resolve) => {
    setTimeout(function () {
      resolve(import("./Article.vue"));
    }, 2000);
  });
});

// 3 秒后加载成功
export const List = defineAsyncComponent(() => {
  return new Promise((resolve) => {
    setTimeout(function () {
      resolve(import("./List.vue"));
    }, 3000);
  });
});
```

```vue
<template>
  <!-- Tab 内容 -->
  <div class="component">
    <component :is="currentComponent"></component>
  </div>
</template>

<script>
  import { Intro, Article, List } from "./components/AsyncComponents.js";
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683626110028-a2d2dd4d-0329-47db-a750-692243956e49.gif)
