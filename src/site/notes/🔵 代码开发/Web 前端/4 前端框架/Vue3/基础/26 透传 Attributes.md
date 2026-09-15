---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/基础/26 透传 Attributes/","dg-note-properties":{}}
---

## 🔢 Attributes 继承

透传 attribute 指的是父组件传递给子组件数据，却没有被子组件声明为 props 或 emits 的 attribute 或者 v-on 事件监听器。最常见的例子就是 class、style 和 id。

当一个组件以单个元素为根作渲染时，透传的 attribute 会自动被添加到根元素上。举例来说，假如我们有一个 <MyButton> 组件，它的模板长这样：

```vue
<template>
  <button>click me</button>
</template>
```

一个父组件使用了这个组件，并且传入了 class：

```vue
<template>
  <MyButton class="large" />
</template>
```

最后的渲染结果是：

```html
<button class="large">click me</button>
```

这里，<MyButton> 并没有将 class 声明为一个它所接受的 prop，所以 class 被视作透传 attribute，自动透传到了 <MyButton> 的根元素上。

## 🔢 对 class 和 style 的合并

如果一个子组件的根元素已经有了 class 或 style 属性，它会和从父组件上继承的值合并。如果我们将之前的 <MyButton> 组件的模板改成这样：

```vue
<template>
  <button class="btn">click me</button>
</template>
```

则最后渲染出的 DOM 结果会变成：

```html
<button class="btn large">click me</button>
```

## 🔢 v-on 监听器继承

同样的规则也适用于 v-on 事件监听器：

```vue
<template>
  <MyButton @click="onClick" />
</template>
```

click 监听器会被添加到 <MyButton> 的根元素，即那个原生的 <button> 元素之上。当原生的 <button> 被点击，会触发父组件的 onClick 方法。

同样的，如果原生 button 元素自身也通过 v-on 绑定了一个事件监听器，则这个监听器和从父组件继承的监听器都会被触发。

## 🔢 深层组件继承

有些情况下一个组件会在根节点上渲染另一个组件。例如，我们重构一下 <MyButton>，让它在根节点上渲染 <BaseButton>：

```vue
<template>
  <BaseButton />
</template>
```

此时 <MyButton> 接收的透传 attribute 会直接继续传给 <BaseButton>。

请注意：

1. 透传的 attribute 不会包含 <MyButton> 上声明过的 props 或是针对 emits 声明事件的 v-on 侦听函数，换句话说，声明过的 props 和侦听函数被 <MyButton>“消费”了。
2. 透传的 attribute 若符合声明，也可以作为 props 传入 <BaseButton>。

## 🔢 禁用 Attributes 继承

如果你不想要一个组件自动地继承 attribute，你可以在组件选项中设置`inheritAttrs: false`。

这种情况适用于我们不想让 attribute 直接应用到组件的根元素上！我们可以通过`$attrs`来指定挂载到我们想要应用的元素上。

```vue
<template>
  <div>
    <button v-bind="$attrs">按钮1</button>
    <button>按钮2</button>
  </div>
</template>

<script>
  export default{
    name: "MyButton",
    inheritAttrs: false
  }
</script>
```

这个`$attrs`对象包含了除组件所声明的 props 和 emits 之外的所有其他 attribute，例如 class，style，v-on 监听器等等。

有几点需要注意：

- 和 props 有所不同，透传 attributes 在 JavaScript 中保留了它们原始的大小写，所以像 foo-bar 这样的一个 attribute 需要通过`$attrs['foo-bar']`来访问。
- 像`@click`这样的一个 v-on 事件监听器将在此对象下被暴露为一个函数`$attrs.onClick`。

## 🔢 多根节点的 Attributes 继承

和单根节点组件有所不同，有着多个根节点的组件没有自动 attribute 透传行为。如果`$attrs`没有被显式绑定，将会抛出一个运行时警告。

```vue
<template>
  <MyButton id="custom-layout" @click="changeValue" />
</template>
```

如果 <MyButton> 有下面这样的多根节点模板，由于 Vue 不知道要将 attribute 透传到哪里，所以会抛出一个警告。

```vue
<template>
  <button>按钮1</button>
  <button>按钮2</button>
</template>
```

我们可以手动的去绑定：

```vue
<template>
  <button v-bind="$attrs">按钮1</button>
  <button>按钮2</button>
</template>
```

另外，我们可以在 script 中访问`$attrs`属性：

```js
export default {
  created() {
    console.log(this.$attrs)
  }
}
```
