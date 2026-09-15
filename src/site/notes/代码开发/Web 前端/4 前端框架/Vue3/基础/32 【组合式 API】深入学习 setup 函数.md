---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/32 【组合式 API】深入学习 setup 函数/","dg-note-properties":{}}
---

`setup()`函数是组合式 API 的入口函数，所有的组合式 API 都需要放到`setup()`函数内部执行。

```js
export default{
  setup(props, context){
    // do...
  }
}
```

`setup()`函数是在`beforCreate()`和`created()`之前执行的，所以`setup()`内部不能调用`beforCreate()`和`created()`这两个生命周期函数，而是使用`setup()`来代替。

Vue3 没有完全去除选项 API，而是做了向下兼容，新增了组合式 API，开发者完全可以自由的选择任意一种或者混合性的编写开发项目。

```js
export default{
  setup(props, { attrs, slots, emit, expose }) {
    console.log(props);

    return {
      msg: "Hello setup!"
    }
  }
}
```

`setup()`函数的第一个参数是`props`，其实就是对选项 API 中`props`的引用。

`setup()`函数的第二个参数是`context`，表示上下文的意思，该对象内部有`attrs`、`slots`、`emit`和`expose`这 4 个对象。

在`setup()`函数中返回的对象会暴露给模板和组件实例。其他的选项也可以通过组件实例来获取`setup()`暴露的属性。

在`setup()`函数内部，是无法像选项 API 那样获取`this`对象的，当访问`this`的时候会得到`undefinde`，你可以在选项式 API 中访问`setup()`暴露的值，但反过来则不行。

为什么无法访问`this`对象呢？

1、因为`setup()`函数是在组件创建之前执行的，执行的时候还没有组件实例，所以也就没有`this`，所以 Vue3 会把一些对象封装作为`setup()`函数的参数。

2、使用组合式 API 编程的时候，一些函数都是通过 ESModule 导入的而不是挂载到实例上的，所以也就没有`this`的必要了。

<br/>

## 🔢 props

```vue
<template>
  <VSetup
    title="This is title."
    author="Xiechen"
    :content="content" />
</template>

<script>
  import VSetup from "./components/VSetup.vue";

  export default {
    components:{
      VSetup
    },
    data() {
      return {
        content: "This is content."
      };
    }
  }
</script>
```

```vue
<template>
  <div class="">
    <h1>{{ title }}</h1>
    <p>{{ author }}</p>
    <p>{{ content }}</p>
  </div>
</template>

<script>
  export default{
    props: {
      title: String,
      author: String,
      content: String
    },
    setup(props, context){
      console.log(props);
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685690901379-2629e2f6-10c1-4821-bf8c-96df757f9c92.png)

你可以完全的把选项 API 和组合 API 混合使用。

`props`是一个响应式的对象，你不能把该对象的属性进行解构使用，这会丢失数据的响应式。

```vue
<script>
  export default{
    mounted() {
      setTimeout(() => {
        this.content = "This is my content.";
      }, 1000);
    }
  }
</script>
```

```vue
<template>
  <div class="">
    <h1>{{ title }}</h1>
    <p>{{ author }}</p>
    <p>{{ content }}</p>
    <p>{{ myContent }}</p>
  </div>
</template>

<script>
  // 把 computed 导入进来
  import { computed } from "vue"

  export default{
    props: {
      title: String,
      author: String,
      content: String
    },
    setup(props, context){
      const { title, author, content } = props;
      const myContent = computed(() => "Content:" + content);

      // setup() 中的数据必须 return 出去，模版才能进行使用
      return {
        myContent
      }
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685691188756-b886fa65-c2aa-4772-a064-14f142cdaca0.gif)

可以看到，模版中的`content`发生了变化，而解构后的`myContent`却没有变。

如果你非要进行解构`props`对象中的属性，可以使用`toRefs`方法进行解构：

```vue
<script>
  import { computed, toRefs } from "vue"

  export default{
    pprops: {
      title: String,
      author: String,
      content: String
    },
    setup(props, context){
      const { title, author, content } = toRefs(props);
      console.log(title, author, content);
      // 因为 content 被包装为 ref 对象，所以需要使用 .value 来获取值
      const myContent = computed(() => "Content:" + content.value);

      return {
        myContent
      };
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685691436260-572ef075-e60f-4048-9f7d-7858b5946ee0.png)

这样得到的属性就全部都是响应式的了。

同样的，如果你只想把一个属性进行解构出来可以使用`toRef()`方法来这么操作：

```vue
<script>
  import { computed, toRefs, toRef } from "vue"

  export default{
    pprops: {
      title: String,
      author: String,
      content: String
    },
    setup(props, context){
      const content = toRef(props, "content");
      const myContent = computed(() => "Content:" + content.value);

      return {
        myContent
      };
    }
  }
</script>
```

## 🔢 context

`context`对象里面包含了`attrs`、`emit`、`slots`、`expose`这 4 个对象。

其中，`attrs`对应着选项 API 中的`this.$attrs`，`slots`对应着选项 API 中的`this.$slots`，`emit`对应着选项 API 中的`this.$emit`。而`expose`是 Vue3 新增的一个方法，这个本篇的后面再说。

```vue
<script>
  export default{
    pprops: {
      title: String,
      author: String,
      content: String
    },
    setup(props, { attrs, slots, emit, expose }) {
      console.log(attrs);
      console.log(slots);
      console.log(emit);
      console.log(expose);
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685692148043-b956d32a-0c54-475c-af99-dccea2f329dd.png)

其中`attrs`和`slots`是对象，`emit`和`expose`是方法。

`attrs`和`slots`都是有状态的对象，它们总是会随着组件自身的更新而更新。这意味着你应当避免解构它们，并始终通过`attrs.x`或`slots.x`的形式使用其中的属性。

此外还需注意，和`props`不同，`attrs`和`slots`的属性都不是响应式的。如果你想要基于`attrs`或`slots`的改变来执行副作用，那么你应该在`onBeforeUpdate`生命周期钩子中编写相关逻辑。

## 🔢 expose

`expose()`是干什么的？

`expose()`函数用于显式地限制该组件暴露出的属性，当父组件通过模板引用访问该组件的实例时，将仅能访问`expose`函数暴露出的内容：

```js
export default {
  setup(props, { expose }) {
    // 让组件实例处于 “关闭状态”
    // 即不向父组件暴露任何东西
    expose()

    const publicCount = ref(0)
    const privateCount = ref(0)
    // 有选择地暴露局部状态
    expose({ count: publicCount })
  }
}
```

`setup()`函数也可以返回一个渲染函数，此时在渲染函数中可以直接使用当前作用域下的响应式状态，但是当`setup()`函数返回一个渲染函数将会阻止我们返回其他东西。这对于组件内部来说，这样没有问题，但如果我们想通过模板引用将这个组件的方法暴露给父组件，那就有问题了。

```html
<!-- 父组件通过插槽的形式调用 -->
<template>
  <VSetup ref="VSetupRef">
    <template #default>This is title.</template>
    <template #author>Xiechen</template>
    <template #content>{{ content }}</template>
  </VSetup>
</template>

<script>
  import { ref, onMounted } from "vue";

  import VSetup from "./components/VSetup.vue";

  export default {
    components: {
      VSetup
    },
    data() {
      return {
        content: "This is content."
      };
    },
    setup() {
      const VSetupRef = ref(null);

      onMounted(() => {
        console.log(VSetupRef.value);
      });

      return {
        VSetupRef
      };
    }
  };
</script>

```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685693952674-1ea2a8c5-38b5-403d-ae35-62a072332345.png)

```vue
<script>
  import { h } from "vue";

  export default {
    setup(props, context) {
      const testStr = "This is test str."

      return () =>
        h("div", null, [
          h("h1", null, context.slots.default()),
          h("p", null, context.slots.author()),
          h("p", null, context.slots.content()),
          h("p", null, testStr)
        ]);
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685693602792-822e9b73-7643-4a1e-85c8-91027a4923d1.png)

就目前的结构来说，父组件是拿不到子组件中的`testStr`属性的。

我们可以通过调用`expose()`解决这个问题：

```vue
<script>
  import { h } from "vue";

  export default {
    setup(props, context) {
      const testStr = "This is test str."

      context.expose({
        testStr
      })

      return () =>
        h("div", null, [
          h("h1", null, context.slots.default()),
          h("p", null, context.slots.author()),
          h("p", null, context.slots.content()),
          h("p", null, testStr)
        ]);
    }
  }
</script>
```

然后父组件就可以拿到这个属性了：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685694065993-015169c8-5791-41b4-8365-04a52418fea4.png)

## 🔢 getCurrentInstance()

在`setup()`函数中，我们可以通过导入`getCurrentInstance()`方法来获取当前组件的实例对象：

```vue
<script>
  import { getCurrentInstance } from "vue";

  export default {
    setup(props, context) {
      console.log(getCurrentInstance());
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1685694169185-6a083489-3757-4279-9c5c-88f522565dc7.png)

<br/>warning
⚠️ 注意

但是 Vue3 不建议在应用代码中进行使用，也不建议使用`getCurrentInstance()`来代替`this`对象！！！

另外，`getCurrentInstance()`函数只能在`setup()`内部执行。

<br/>
