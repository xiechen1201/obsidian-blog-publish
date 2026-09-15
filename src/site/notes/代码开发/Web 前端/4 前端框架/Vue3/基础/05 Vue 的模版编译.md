---
{"dg-publish":true,"permalink":"//web/4/vue3//05-vue/","dg-note-properties":{}}
---

`Vue`使用一种基于`HTML`的模板语法，使我们能够声明式地将其组件实例的数据绑定到呈现的`DOM`上。所有的`Vue`模板都是语法层面合法的`HTML`，可以被符合规范的浏览器和`HTML`解析器解析。

`template`内部的`HTML`字符串，内部有一些`Vue`的特性，例如插值表达式、自定义属性和指令。

`Vue`提供一套模版编译系统来编译`Vue`的模版语法，解析流程如下：

开发者写的`template`

➡️ 分析`HTML`字符串

➡️ 生成`AST`树

➡️ 解析表达式、自定义属性、指令

➡️ 生成虚拟`DOM`

➡️ 解析为真实的`DOM`

➡️ `render`渲染到页面上

<br/>

为什么需要虚拟`DOM`呢？

[虚拟 DOM](https://www.yuque.com/xiechen/de54wv/nqlw0gyg8sv9hn5d)

除了使用`template`,`Vue`还支持`render`+`h`函数：

```js
import { createApp, h } from "vue";

const App = {
  render() {
    return h("div", {}, [
      h("h1", { class: "title" }, this.title),
      h("p", {}, "This is content")
    ]);
  },
  data() {
    return {
      title: "this is my title",
    };
  },
};

createApp(App).mount("#app");
```

## 🔢 插值表达式

插值表达式内只能书写`JS`表达式，不能书写语句、函数、声明等！

```js
const App = {
  template: `
  	<h1 class="title">{{ title }}</h1>
  	<p>{{ var a = 1 }}</p> 这是一个语句，而非表达式
  	<p>{{ if (ok) { return message } }}</p> 条件控制也不支持
  `,
  data() {
    return {
      title: "this is my title",
    };
  },
};
```

插值表达式的想法来源于`Mustache`八字胡。

[GitHub - janl/mustache.js: Minimal templating with {{mustaches}} in JavaScript](https://github.com/janl/mustache.js)

我们可以利用这个库实现一个简单的案例：

```js
import mustache from "mustache";

let data = {
  title: "This is My Title",
};
let html = mustache.render(`<h1>{{ title }}</h1>`, data);

console.log(html); // <h1>This is My Title</h1>
```

以上案例我们利用`Mustache`就将`{{}}`内的数据进行了替换，但是`Vue`并没有使用`Mustache`库，只是灵感来源于它。

`Mustache`并不支持`HTML`书写的插值，`Vue`的底层有一套自己的模版编译系统，所以能支持`Vue`内置的属性。

## 🔢 Vue 的内置指令

在`Vue`开发中，所有书写在`Vue`模版中`v-*`都是指令。为什么叫做指令呢？指令可以告诉模版应该按照什么样的逻辑进行渲染或绑定行为。

`Vue`提供了大量的内置指令，如`v-if`、`v-show`、`v-for`、`v-html`...同时`Vue`还支持我们给`Vue`拓展指令（也就是自定义指令）。

这里我们挑几个指令讲述一下使用时的注意点：

- `v-html`，在插值中是不会解析`HTML`字符串的，因为插值是`JS`表达式，没有对`DOM`的操作。

```vue
<template>
  <div>{{ '<p>'+ 内容 +'</p>' }}</div>
</template>

<!-- 编译结果为：<p>内容</p> -->
```

不要试图将`v-html`作为子模版的显示，因为`Vue`本身有一套底层的模版编译系统，而不是直接使用字符串进行渲染，这会导致语法无法被解析，你应该把子模版放到子组件中，让模版的重用和组合更加强大。

```vue
<template>
  <!-- 不要这样做 -->
  <div v-html="child"></div>
</template>

<script>
  export default{
    data(){
      return{
        child: `<button @click="handleClickBtn">点击按钮</button>`
      }
    }
  }
</script>
```

另外需要注意，`v-html`是动态渲染的`HTML`，使用的基本都是`innerHTML`属性进行赋值，`innerHTML`很容易导致`XSS`的攻击！！！

```js
let text = "<img src="123" onerror="alert(123)" />" // 图片加载失败将会执行 alert
document.body.innerHTML = text;
```

- `v-once`用于让`template`一次插值后，再也不更新，这个指令同时会影响到子组件！！！

```vue
<template>
  <h1 v-once>{{ title }}</h1>
  <button @click="handleClick">更改title</button>
</template>

<script>
  export default {
    data() {
      return {
        title: "this is title",
      };
    },
    methods: {
      handleClick() {
        // title 属性变化后，h1 标签不会更新
        this.title = new Date().getTime();
      },
    },
  };
</script>
```

- `v-bind`用于动态绑定属性，布尔型的属性依据`true/false`值来决定`attribute`是否应该存在于该元素上。`disabled`就是最常见的例子之一。

当`isButtonDisabled `为真值或一个空字符串 (即`<button disabled="">`) 时，元素会包含这个`disabled attribute`。而当其为其他假值时`attribute`将被忽略。

```js
<button :disabled="isButtonDisabled">Button</button>
```

如果你有像这样的一个包含多个`attribute`的`JavaScript`对象，通过不带参数的`v-bind`，你可以将它们绑定到单个元素上：

```js
<template>
  <div v-bind="objectOfAttrs"></div>
  <!-- 和下面的等效的 -->
  <div v-bind:id="objectOfAttrs.id" v-bind:class="objectOfAttrs.class"></div>
</template>

<script>
  export default{
    data(){
      return{
        objectOfAttrs:{
          id: 'container',
          class: 'wrapper'
        }
      }
    }
  }
</script>
```
