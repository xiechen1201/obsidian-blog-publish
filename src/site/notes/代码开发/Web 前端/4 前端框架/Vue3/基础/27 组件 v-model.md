---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/27 组件 v-model/","dg-note-properties":{}}
---

v-model 可以在组件上使用实现双向绑定。

首先让我们回忆一下 v-model 在原生元素上的用法：

```html
<input v-model="searchText" />
```

在代码背后，模板编译器会对 v-model 进行更冗长的等价展开。因此上面的代码其实等价于下面这段：

```html
<input
  :value="searchText"
  @input="searchText = $event.target.value"
/>
```

而当使用在一个组件上时，v-model 会被展开为如下的形式：

```html
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
```

要让这个例子实际工作起来，`<CustomInput>`组件内部需要做两件事：

1. 将内部原生`<input>`元素的 value 属性绑定为`modelValue`；
2. 当表单的`input`事件触发时，触发一个携带了新值的`update:modelValue`自定义事件；

```vue
<!-- CustomInput.vue -->
<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
    />
</template>

<script>
  export default {
    props: ['modelValue'],
    emits: ['update:modelValue']
  }
</script>
```

现在`v-model`可以在这个组件上正常工作了：

```html
<CustomInput v-model="searchText" />
```

## v-model 的参数

默认情况下，`v-model`在组件上都是使用 modelValue 作为 prop，并以`update:modelValue`作为对应的事件。我们可以通过给`v-model`指定一个参数来更改这些名字：

```html
<MyComponent v-model:title="bookTitle" />
```

在这个例子中，子组件应在`props`中注册`title`属性，并通过触发`$emit`发送一个名为`update:title`的事件更新父组件值：

```vue
<!-- MyComponent.vue -->
<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>

<script>
export default {
  props: ['title'],
  emits: ['update:title']
}
</script>
```

## 多个 v-model 绑定

利用刚才在 v-model 参数小节中学到的指定参数与事件名的技巧，我们可以在单个组件实例上创建多个 v-model 双向绑定。

组件上的每一个 v-model 都会同步不同的 prop，而无需额外的选项：

```html
<!-- 父组件 -->
<UserName
  v-model:first-name="first"
  v-model:last-name="last"
/>
```

```vue
<!-- 子组件 -->
<template>
  <input
    type="text"
    :value="firstName"
    @input="$emit('update:firstName', $event.target.value)"
  />
  <input
    type="text"
    :value="lastName"
    @input="$emit('update:lastName', $event.target.value)"
  />
</template>

<script>
export default {
  props: {
    firstName: String,
    lastName: String
  },
  emits: ['update:firstName', 'update:lastName']
}
</script>
```

## 处理 v-model 修饰符

在学习输入绑定时，我们知道了 v-model 有一些内置的修饰符，例如`.trim`、`.number`和`.lazy`。在某些场景下，你可能想要一个自定义组件的 v-model 支持自定义的修饰符。

我们来创建一个自定义的修饰符`.prefixer`，它会自动将 v-model 绑定输入的字符串值拼接上“UP主：”

```html
<my-input v-model.prefixer="myName" />
```

组件的 v-model 上所添加的修饰符，可以通过`modelModifiers`在组件内访问到。

在下面的组件中，我们声明了`modelModifiers`这个属性，它的默认值是一个空对象：

```html
<template>
  <div>
    <h1>{{ modelValue }}</h1>
    <input type="text" :value="modelValue" @input="emitName" />
  </div>
</template>

<script>
  export default {
    name: "MyInput",
    props: {
      modelValue: String,
      modelModifiers: {
        default: () => ({})
      }
    },
    emits: ["update:modelValue"],
    created() {
      console.log(this.modelModifiers); // { prefixer: true }
    },
    methods: {
      emitName(e) {
        let inputValue = e.target.value;

        // 我们可以判断是否有 prefixer 这个属性来决定是否要拼接 'UP主：'
        if (this.modelModifiers.prefixer && !inputValue.match(/UP主：/)) {
          inputValue = "UP主：" + inputValue;
        }
        this.$emit("update:modelValue", inputValue);
      }
    }
  };
</script>

```

对于又有参数又有修饰符的 v-model 绑定，生成的 prop 名将是`arg + Modifiers`。举例来说：

```html
<my-input v-model:my-name.prefixer="myName" />
```

相应的声明是：

```js
export default {
  props: ['myName', 'myNameModifiers'],
  emits: ['update:myName'],
  created() {
    console.log(this.myNameModifiers) // { prefixer: true }
  }
}
```
