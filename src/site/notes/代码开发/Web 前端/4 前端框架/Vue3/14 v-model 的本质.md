---
{"dg-publish":true,"permalink":"//web/4/vue3/14-v-model/","dg-note-properties":{}}
---

`v-model`的用法，总结起来就两个场景：

1、表单元素和响应式数据的双向绑定；

2、父子组件传递数据；

## 🔢 和表单元素的绑定

```vue
<template>
  <div>
    <p>输入的内容为：{{ message }}</p>
    <input v-model="message" type="text" placeholder="请输入内容" />
  </div>
</template>

<script setup>
import { ref } from 'vue';

const message = ref('hello world');
</script>
```

在这个案例中，`<input />`元素和响应式数据`message`做了双向的绑定。当我们在`<input />`元素内输入内容的时候，`message`数据的值也会被更改，反过来当我们更改`message`值的时候，`<input />`元素的内容也会产生变化。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1728369074041-8d920847-4a23-42b1-992e-653f40e95111.gif)

## 🔢 和子组件进行绑定

```vue
<template>
  <div class="app-container">
    <h1>请打分：</h1>
    <RatingComponent v-model="rating" />
    <p>您的评分：{{ rating }} / 5</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';

import RatingComponent from './components/RatingComponent.vue';

const rating = ref(3);
</script>
```

```vue
<template>
  <div>
    <span
      v-for="star in 5"
      :key="star"
      class="star"
      @click="setRating(star)">
      {{ model >= star ? '★' : '☆' }}
    </span>
  </div>
</template>

<script setup>
// 接收父组件传递过来的 v-model 属性
const model = defineModel();

// 更新父组件中的 v-model 属性
function setRating(star) {
  model.value = star;
}
</script>
```

在本案例中，父组件通过`v-model`讲自身的数据传递给子组件，子组件通过`defineModel()`编译宏来拿到父组件传递过来的数据。拿到数据之后不仅可以进行使用，还可以进行更改。

<br/>tips
🔔 提示

从 Vue3.4 版本开始，官方推荐的实现方式是使用`[defineModel()](https://cn.vuejs.org/api/sfc-script-setup.html#definemodel)`宏。

<br/>

下面是 Vue3.4 版本之前的写法：

```vue
<script setup>
// 接收 modelValue 作为 prop
const props = defineProps(['modelValue']);
// 定义 emit 来触发父组件的更新
const emit = defineEmits(['update:modelValue']);

function setRating(star) {
  // 更新父组件中的 modelValue
  emit('update:modelValue', star);
}
</script>
```

## 🔢 v-model 的本质

首先我们先分析第一个场景：和表单元素进行绑定。

> 同样使用 vite-plugin-inspect 插件查看编译的结果。
>

查看编译的结果：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1728369833612-aa6e6f8a-fb3e-41c7-8e7a-5eee34f18e26.png)

根据编译结果可以看出，`<input />`元素上的`v-model`会被展开为一个名为`onUpdate:modelValue`的自定义事件，该事件对应的处理函数是`$event => (($setup.message) = $event)`。

这就解释了输入框输入值的时候，为什么会影响响应式数据的值。而输入框的`value`本身又是和`$setup.message`绑定在一起的，`$setup.message`一变化就会导致「渲染函数」重新运行，从而看到输入框里面的内容发生了变化。

接下来分析第二个场景：和子组件使用`v-model`进行绑定。

父组件：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1728370993942-c4bce2f8-ca7a-4689-aa1b-06c538fb3a68.png)

父组件会向子组件传递一个名为`modelValue`的`props`，`props`对应的值就是`$setup.rating`，这真是父组件上面的状态。

除此之外，还传递了一个名为`onUpdate:modelValue`的自定义事件，该事件对应的事件处理函数：`$event => (($setup.rating) = $event)`。该事件处理函数负责的事情就是将接收到的值更新组件本身的数据`rating`。

子组件：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1728373751862-62871aca-9b4f-4ab9-8bf3-366b92960d56.png)

对于子组件来说，就可以通过`modelValue`这个`props`来拿到父组件传递过来的数据，并且在模版中使用该数据。

当更新数据的时候，就会触发父组件传递过来的`onUpdate:modelValue`事件，并且将新的值传递过去。

到这里，你对官网下面的这句话就明白了：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1728373928932-c175e336-31a0-430b-888a-7fe780d92663.png)

某些时候，我们可以在子组件上使用具名的`v-model`，此时展开的`props`和自定义的事件名称就会有所不同。

父组件：

```vue
<template>
  <div class="app-container">
    <h1>请打分：</h1>
    <!-- 这是一个具名的 v-model -->
    <RatingComponent v-model:title="rating" />
    <p>您的评分：{{ rating }} / 5</p>
  </div>
</template>
```

编译结果：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1728374105629-19ce71d9-ba5a-4180-89b1-b03dd5e44a63.png)

可以看到，当给`v-model`指定名称后，之前的`modelValue`就变成了`title`。自定义的`onUpdate:modelValue`也变成了`onUpdate:title`。

子组件：

```vue
<script setup>
// 接收父组件传递过来的 mode 属性
// 这里获取状态的时候就需要指定一下名字
const model = defineModel("title")

function setRating(star) {
  model.value = star
}
</script>
```

编译结果：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1728374318664-af99eb31-7ab0-4cb7-b29b-ae132662cc61.png)
