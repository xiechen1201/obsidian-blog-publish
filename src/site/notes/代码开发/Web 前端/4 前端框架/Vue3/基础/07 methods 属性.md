---
{"dg-publish":true,"permalink":"//web/4/vue3//07-methods/","dg-note-properties":{}}
---

在`Vue`中`methods`属性用于书写实例的事件处理方法，`Vue`在创建实例的时候会自动把`methods`绑定到当前实例的`this`中，写在`mthods`对象中的方法要避免使用「箭头函数」，箭头函数可能会影响到`Vue`正确的指向`Vue`的实例`this`。

```js
let app = {
  template: `
    <h1>{{ title }}</h1>
    <button v-on:click="changeTitle('title')">change title</button>
  `,
  data() {
    return {
      title: "This is my title.",
    };
  },
  methods: {
    // 不推荐
    changeTitle: () => {
      this.title = "This is your title.";
    },
  },
};
```

在`Vue`的模版中，函数名后面紧挨着`()`并不是执行符号，`()`内可以传递你想要的参数，这个过程会被编译为`v-on:click="()=> changeTitle('title')"`。

```html
<button v-on:click="changeTitle('title')">change title</button>
<!-- v-on:click="() => changeTitle('title')" -->
```

在`Vue`的实例中，虽然挂载了`methods`对象里面的方法，但是并没有像`$data`一样被抛出了，这是因为`data`属性被更改的时候`$data`也会被更改，所以必须要暴露出来，而`methods`本身就是一些方法的集合容器，直接挂载到实例上能够调用就可以了，没必要暴露出来。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673321065546-b75d82c3-855c-44b4-8495-fdfb1ffe5668.jpeg)
