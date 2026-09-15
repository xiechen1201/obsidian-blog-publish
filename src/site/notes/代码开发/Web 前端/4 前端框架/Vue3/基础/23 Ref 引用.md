---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/23 Ref 引用/","dg-note-properties":{}}
---

`ref`主要是引用 DOM 节点或者引用组件实例，说白了就是 Vue 本身不需要你操作 DOM，Vue 底层已经帮你做好了数据的绑定，所有视图的更新都来源于 viewModel 去做双向数据绑定。但是，某些时候你又不得不去获取 DOM 节点（或者 DOM 的某些信息），或者你要操作组件的实例，这个时候你就可以使用`ref`去引用！

## ref 引用 DOM 元素

当我们给 DOM 元素设置`ref`属性的时候，我们就可以在`this.$refs`中拿到 DOM 节点的引用。

```vue
<template>
	<p>
		<label for="">Username:</label>
		<input type="text" ref="myRef" name="" id="" />
	</p>
</template>

<script>
	export default{
		mounted(){
			console.log(this.$refs);
		}
	}
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683792569854-f560c5fe-0146-4131-abdf-9f2c9cd4f309.png)

但其实，组件实例在`beforeCreate`阶段就已经能拿到`$refs`对象，只不过是个空对象，直到`mounted`组件挂载后，`$refs`对象才能拿到 DOM 的引用！

```vue
<template>
  <p>
    <label for="">Username:</label>
    <input type="text" ref="myRef" name="" id="" />
  </p>
</template>

<script>
  export default {
    beforeCreate() {
      console.log(this.$refs);
    },
    created() {
      console.log(this.$refs);
    },
    beforeMount() {
      console.log(this.$refs);
    },
    mounted() {
      console.log(this.$refs);
    }
  };
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683792684657-b999a629-1c57-4292-abc7-4482fbf69ac7.png)

这主要是因为 DOM 节点还没有挂载，所以你并不能拿到引用。

如果你尝试修改`$refs`对象的属性，你讲看到警告信息，因为`$refs`对象属性是只读的。

```js
export default {
  mounted() {
    const oLink = document.createElement("a");
    oLink.innerText = "Google";
    oLink.href = "https://www.google.com";
    // 不可进行修改
    this.$refs.myRef = oLink;
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683792815960-d8952c2c-c966-4943-954d-c9535ed6c504.png)

因为`ref`是在 DOM 渲染后才进行挂载，所以当你通过`v-if`快速切换`ref`的时候，你将看到非理想的的结果：

```vue
<template>
  <p v-if="showWhat === 'input'">
    <label for="">Username:</label>
    <input type="text" ref="myRef" name="" id="" />
  </p>
  <p v-else-if="showWhat === 'link'">
    <a href="https://www.google.com" ref="myRef">Google</a>
  </p>
</template>

<script>
  export default {
    data() {
      return {
        showWhat: "link"
      };
    },
    mounted() {
      console.log(this.$refs);
      this.showWhat = "input";
      console.log(this.$refs);
    }
  };
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683793084105-dd02aabf-739a-46e8-abcb-a205de38f6fd.png)

你将看到两个`$refs.myRef`都为`a`！

如果想要避免这样的情况发生，你应该使用`setTimeout`进行延迟获取，或者使用 Vue 的`nextTick`来回调获取：

```js
export default {
  data() {
    return {
      showWhat: "link"
    };
  },
  mounted() {
    console.log(this.$refs);
    this.showWhat = "input";
    setTimeout(() => {
      console.log(this.$refs);
    });

    // 或者
    console.log(this.$refs);
    this.showWhat = "input";
    this.$nextTick(() => {
      console.log(this.$refs);
    });
  }
};
```

当我们获取到 DOM 节点的时候，我们就可以对 DOM 进行操作或获取 DOM 的信息：

```js
export default {
  mounted() {
    this.$refs.myRef.focus();
    this.$refs.myRef.offsetHeight;
  }
};
```

## ref 引用组件实例

`ref`还可以用来引用组件实例：

```html
<my-test ref="myTestRef"></my-test>
```

当`ref`作用在组件上的时候，是可以拿到组件的实例，而 DOM 则是渲染后的 DOM 节点。

和 DOM 一样，`ref`引用组件也是在渲染完成后才能获取到。

```js
export default {
  beforeCreate() {
    console.log(this.$refs);
  },
  created() {
    console.log(this.$refs);
  },
  mounted() {
    console.log(this.$refs);
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683793686763-dbf16fc3-f66b-4925-b20e-4a00c1df450e.png)

另外，`ref`本身并不是响应式的，所以不要在模版、计算属性（因为非响应式不会触发`computed`重新计算）中进行使用，也不要尝试去修改`ref`的值（`ref`只提供你获取 DOM 或组件，并不是让你去操作的！）

```js
export default{
  mounted(){
    // 不要这么做
    this.$refs.myTestRef.count++;
  }
}
```

通过`ref`可以获取到组件内部的属性或者方法，但大多数情况下，你应该首先使用标准的`props`和`emit`接口来实现父子组件交互！

```vue
<template>
  <my-test ref="myTestRef"></my-test>
</template>

<script>
  import MyTest from "./components/MyTest/index.vue";

  export default {
    components: {
      MyTest
    },
    methods: {
      handleClick() {
        // 可以获取到 myTestRef.count 属性
        console.log(this.$refs.myTestRef.count);

        // 可以调用 myTestRef.addCount 方法
        this.$refs.myTestRef.addCount();
      }
    }
  };
</script>
```

```vue
<template>
  <button @click="handleLog">点击</button>
</template>

<script>
  export default {
    name: "Test",
    data() {
      return {
        count: 0
      };
    },
    methods: {
      handleLog() {
        this.addCount();
      },
      addCount() {
        this.count++;
        console.log(this.count)
      }
    }
  };
</script>
```

## v-for 中的模板引用

当在 v-for 中使用模板引用时，相应的引用中包含的值是一个数组：

```vue
<template>
  <ul>
    <li v-for="item in 10" ref="items">
      {{ item }}
    </li>
  </ul>
</template>

<script>
  export default {
    mounted() {
      console.log(this.$refs.items)
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683794602733-3907757c-74c3-4a09-a6c3-074ca0331e1b.png)

<br/>warning
⚠️ 注意

应该注意的是，ref 数组并不保证与源数组相同的顺序。

<br/>
