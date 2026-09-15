---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/29 mixins 混入/","dg-note-properties":{}}
---

mixins 是 Vue2 OptionsAPI 的产物，在 Vue3 的版本中已经不再推荐使用了。

mixins 应用场景是当两个或多个组件的某些数据、逻辑相同的时候，可以把这些相同的数据、逻辑抽离为一个单独的 JS 文件，该 JS 文件内和 SFC 文件中的`<script>`结构是一样的，都是导出一个模块。

```js
export default{
  // 共用的 data 数据
  data(){
    return{
      title: "This is test title."
    }
  },
  // 公共的逻辑处理
  methods:{
    onCliclAdd(){
      console.log("This is test methods.")
    }
  }
}
```

然后我们在需要复用的组件内部通过`mixins`属性进行注册，就实现了混入！

```vue
<template>
  <!-- 可以直接调用 test.js 混入的方法 -->
  <button @click="onCliclAdd">Add btn</button>
</template>

<script>
  import testMixin from "./mixins/test.js";

  export default{
    name:"App",
    // 通过数组的方式注册，所以一个组件可以注册多个 mixin
    mixins: ['testMixin'],
    mounted(){
      // 可以直接调用混入文件的数据
      console.log(this.title);
    }
  }
</script>
```

当然，mixins 和 component 一样，可以进行全局注册，但这不是被推荐的：

```js
import { createApp } from "vue";
import App from "./App.vue";
import testMixin from "./mixins/test"

const app = createApp(App);
app.mixin(testMixin);
app.mount("#app");
```

因为混入文件的数据、逻辑会和普通组件内的数据、逻辑混合在一块，所以混入的时候会存在对应的规则：

1、当组件内部有的数据，mixins 不会进行覆盖。

也就是说，当选项中的 data 有冲突的时候，组件内的 data 优先！

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>{{ author }}</p>
    <p>{{ content }}</p>
  </div>
</template>

<script>
import mixinTest from "../mixins/test";

export default {
  name: "App",
  mixins: [mixinTest],
  data() {
    return {
      title: "This is test 1.",
      content: "This is content 1."
    };
  }
};
</script>
```

```js
export default {
  data() {
    return {
      title: "This is mixin title.",
      author: "This is mixin author.",
      content: "This is mixin content."
    };
  }
};
```

结果如下：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1684909619368-0c6d54b2-dfc9-4d05-84a7-9fa174badda4.png)

优选使用组件内部的 title 和 content，组件内部没有 author 才会去使用 mixins 文件的 author。

2、当组件和 mixins 文件有相同钩子函数的时候，优先执行 mixins 的钩子函数。

```vue
<script>
import mixinTest from "../mixins/test";

export default {
  name: "App",
  mixins: [mixinTest],
  mounted() {
    console.log("This is test1 mounted calling.");
    console.log(this);
  }
};
</script>
```

```js
export default {
  mounted() {
    console.log("This is Mixin mounted calling.");
  }
};
```

结果如下：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1684909840237-c9e9ea27-57e1-4d30-b13e-0a151dafebba.png)

3、mixins 文件内的相关 options 会合并到组件实例对象上！

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>{{ author }}</p>
    <p>{{ content }}</p>
    <button @click="doComponent">btn</button>
  </div>
</template>

<script>
import mixinTest from "../mixins/test";

export default {
  name: "App",
  mixins: [mixinTest],
  data() {
    return {
      title: "This is test 1.",
      content: "This is content 1."
    };
  },
  mounted() {
    console.log("This is test1 mounted calling.");
    console.log(this);
  },
  methods: {
    doComponent() {
      console.log("doComponent");
    }
  }
};
</script>
```

```js
export default {
  data() {
    return {
      title: "This is mixin title.",
      author: "This is mixin author.",
      content: "This is mixin content."
    };
  },
  mounted() {
    console.log("This is Mixin mounted calling.");
  },
  methods: {
    doMixin() {
      console.log("doMixin");
    }
  }
};
```

结果如下：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1684910038334-28e68591-f157-4b70-b344-e15a70d8db94.png)

会把 mixins 内的 data、methods 合并到组件实例上面。

4、当组件内和 mixins 文件有同名方法时候，优先执行组件内部的方法。

```vue
<template>
  <div>
    <!-- 省略其他内容 -->
    <button @click="doComponent">btn</button>
  </div>
</template>

<script>
  import mixinTest from "../mixins/test";

  export default {
    name: "App",
    mixins: [mixinTest],
    methods: {
      doComponent() {
        console.log("doComponent");
      }
    }
  };
</script>
```

```js
export default {
  methods: {
    doComponent() {
      console.log("doMixinComponent");
    }
  }
};
```

结果如下：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1684910186161-65bb2cbf-c7fa-4ac4-943e-acb72b06fada.png)

mixins 的缺点：

1、当一个 mixins 文件被多个组件注册的时候，可能会多出很多非必要的选项或者属性，可能为了满足需求，导致把 mixin 文件进行无限的拆分，也有可能导致命名冲突；

2、因为 mixins 导出的是一个对象，没有办法通过函数那样通过参数去控制哪些参数是需要的，哪些是不需要的，这极大的干扰了 mixins 的合理性复用，所以 Vue3 不建议使用，而是使用 CompositionAPI。所有需要复用、集成的功能全部封装为函数，该函数内部可以使用 Vue3 提供的相关 API，例如 watch、ref 等等...
