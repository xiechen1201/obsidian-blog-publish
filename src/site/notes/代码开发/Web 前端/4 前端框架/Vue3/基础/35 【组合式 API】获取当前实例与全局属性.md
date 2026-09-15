---
{"dg-publish":true,"permalink":"//web/4/vue3//35-api/","dg-note-properties":{}}
---

在 Vue2 的时候我们如果想要设置一个全局数据通常都是在 Vue 的构造函数原型上增加属性：

```js
import Vue from "vue";
import App from "./App.vue";

Vue.prototype.utils = {
  $http: function(){}
};

const app = new Vue({
  render: (h) => h(App)
}).$mount("#app");

console.log(app);
```

这样就把`$http`属性挂载到了 Vue 的构造函数上，又因为 Vue 的组件都是 Vue 构造函数的实例，所以组件内可以通过`this`访问组件的实例，顺着原型链就可以拿到`$http`。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686123726375-50121210-ce2a-4903-863f-b63642bc3b17.png)

```js
export default {
  mounted(){
    // 可以获取到且正常执行！
    this.$http();
  }
}
```

到了 Vue3 因为构造应用实例的方式不再使用 new 实例化的形式，所以 Vue3 提供了`app.config.globalProperties`让我们挂载全局的数据。

```js
import { createApp } from "vue";
import App from "./App.vue";

const app = createApp(App);
app.config.globalProperties.$http = function () {};
app.mount("#app");
```

如果你在 Vue3 中使用选项式 API 仍然是可以通过`this`对象来访问到`$http`:

```js
export default {
  mounted(){
    console.log(this.$http);
  }
};
```

但如果你使用组合式 API 就不能使用`this`，而是使用`getCurrentInstance()`来获取当前组件的实例：

```js
export default {
  setup() {
    const instance = getCurrentInstance();
    console.log(instance.proxy.$http);
  }
};
```

<br/>warning
⚠️ 注意

虽然`getCurrentInstance()`能获取到组件的实例对象，但是 Vue 官方是不推荐我们使用的！！！所以更不要使用`getCurrentInstance()`来替代选项式 API 中的`this`！！！

<br/>

但是你会有疑问，能使用`getCurrentInstance()`获取到全局的数据，但是官方又不推荐使用，这不是很矛盾吗？

这其实`app.config.globalProperties`本身就不是给组合式 API 使用！！！

如果你真的使用组合式 API 又想使用全局数据，就可以使用 Vue 的依赖注入来实现：

```js
import { createApp } from "vue";
import App from "./App.vue";
import globalProperties from "./globalProperties"

const app = createApp(App);
app.use(globalProperties)
// app.config.globalProperties.$http = function () {};

app.mount("#app");
```

```js
export default {
  install(app) {
    app.provide("$http", function () {});
  }
};
```

然后需要使用的地方使用`inject()`方法把数据进行注入：

```js
import { inject } from "vue";

export default {
  components: {
    GrandFather
  },
  setup() {
    // const instance = getCurrentInstance();
    // console.log(instance.proxy.$http);

    // 正常执行！！！
    const $http = inject("$http")
    console.log($http);
  }
};
```
