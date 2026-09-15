---
{"dg-publish":true,"permalink":"//web/4/vue3//24-props/","dg-note-properties":{}}
---

说到 props 就不得不提到「单向数据流」，单向数据流是一种组件化中的数据流向的规范，数据总是从父组件流向子组件，这遵循了子组件不能更改父组件流入的数据，这个数据就是 props ！

子组件为什么不可以更改 props 呢？

这是因为如果某个数据属于父组件定义的（父组件传递给子组件的是一个数据的副本，尤其是引用数据的时候），当子组件去更改父组件传递来的 props ，首先父组件的数据将会受到影响，其次这会导致概念上的越权，且不符合权限上的规范。

另外当父组件内包含很多子组件的时候，父组件是不知道哪个子组件更改 data 中的数据，这会造成很大的维护的成本！

## 🔢 props 基本传递

当父组件调用子组件的时候，可以给子组件传递任意数量、任意类型的数据：

```vue
<template>
  <div>
    <!-- 默认传递的都是字符串类型 -->
    <my-test num="1" arr="[1,2,3,4,5]" />
  </div>
</template>
```

子组件只需要通过 props 属性注册一下父组件传递来的数据即可，这样子组件才知道哪些是外部传递来的数据：

```vue
<template>
  <div>
    {{ num }}
    {{ arr }}
  </div>
</template>

<script>
  export default{
    name:"MyTest",
    props: ['num','arr']
  }
</script>
```

如果你想传递特定的数据类型，需要使用 v-bind 或者 : 进行动态绑定：

```vue
<template>
  <div>
    <my-test v-bind:num="1" :arr="[1,2,3,4,5]" :str="'123'" />
  </div>
</template>
```

此时，引号内部传递的就是 JS 表达式，而不再是一个普通的字符串！

或者你也可以传递 data 中的数据：

```vue
<template>
  <div>
    <my-test :num="num" :arr="arr" :str="str" />
  </div>
</template>

<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    data(){
      return{
        num: 1,
        arr: [1, 2, 4],
        str: "123"
      }
    }
  }
</script>
```

如果你想一次性传递多个属性，你可以使用 v-bind 指令批量传递：

```vue
<template>
  <div>
    <my-test v-bind="obj" />
    <!-- 等同于 -->
    <!-- <my-test :a="obj.a" :b="obj.b" /> -->
  </div>
</template>

<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    data(){
      return{
        obj: {
          a: 1,
          b: 2
        }
      }
    }
  }
</script>
```

```vue
<template>
  <div>
    {{ a }}
    {{ b }}
  </div>
</template>

<script>
  export default{
    name:"MyTest",
    props: ['a','b']
  }
</script>
```

当父组件传递数据后，子组件可用通过`props: [xxx]`的方式来进行接收注册，但是通过数组的方式，子组件无法验证父组件传递来的数据是否符合类型要求，Vue 支持把 props 改为一个对象的方式来进行注册属性：

```vue
<template>
  <div>
    {{ a }}
    {{ b }}
  </div>
</template>

<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    props: {
      num: Number,
      arr: Array,
      str: String,
      bool: Boolean,
      obj: Object,
      setNum: Function
    }
  };
</script>
```

当父组件传递的数据不符合 props 的类型要求时，将抛出一个警告：

```vue
<template>
  <div>
    <!-- 子组件验证 num 为 Number 类型 -->
    <my-test :num="{}" />
  </div>
</template>

<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    data(){
      return{
        obj: {
          a: 1,
          b: 2
        }
      }
    }
  }
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1684143016315-a05d4f1f-7df8-4a78-8816-c10b32246616.jpeg)

Vue 之所以抛出警告，而不是错误是因为类型校验本身就是方便开发者去检查类型是否符合要求。

当你不确定一个数据是什么类型的时候，你可以使用数组来指定`props.x`为多种类型：

```vue
<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    props: {
      title: String,
      isRecom: Boolean,
      status: [String, Number],
      author: Object,
      content: String,
      commentCount: Number,
      comments: Array
    }
  };
</script>
```

你可能注意到`setNum`属性是一个函数类型，是的，这是容许的！

```vue
<template>
  <div>
    <!-- setNum 为 Function 类型 -->
    <my-test setNum="setNum" />
  </div>
</template>

<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    methods:{
      setNum(){
        console.log("Parent component function is call!")
      }
    }
  }
</script>
```

```vue
<template>
  <!-- 点击的时候会正常执行父组件中的 setNum 方法  -->
  <button @click="setNum">Click btn</button>
</template>

<script>
  export default {
    name: "MyTest",
    props: {
      setNum: Function
    }
  };
</script>
```

实际上，`props.x`的类型是相应类型的构造函数，所以我们可以自定一个构造函数，让 Vue 进行校验：

```vue
<template>
  <!-- 传递一个类型为 Demo 的对象 -->
  <my-test :demo="new Demo()" />
</template>

<script>
  import { Demo } from "./myFunction.js";
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    methods:{
      Demo
    }
  }
</script>
```

```vue
<template>
  {{ demo }}
</template>

<script>
  import { Demo } from "../myFunction.js";

  export default {
    name: "MyTest",
    props: {
      demo: Demo // 这将正常通过校验
    }
  };
</script>
```

## 🔢 深入 props 的应用

props 也可以设置默认值，当父组件没有传递对应属性的时候生效：

```vue
<template>
  <div>
    {{ num }}
  </div>
</template>

<script>
  export default {
    name: "MyTest",
    props: {
      // 通过对象的形式进行表达
      num: {
        type: Number,
        default: 100
      }
    }
  };
</script>
```

<br/>warning
⚠️ 注意

如果声明了 default 值，那么在 prop 的值被解析为 undefined 时，无论 prop 是未被传递还是显式指明的 undefined，都会改为 default 值。

<br/>

如果你需要给一个对象设置默认值，应该通过函数返回一个对象，这和组件内 data 返回一个对象道理是一致的：

```vue
<script>
  export default {
    name: "MyTest",
    props: {
      article: {
        type: Object,
        default: ()=>({})
      }
    }
  };
</script>
```

但是当时类型为函数的时候，默认值却不同！当类型为函数的时候，default 的值将作为函数的默认值：

```vue
<template>
  <!-- 点击按钮就会打印 'Default function' -->
  <button @click="setNum">Click btn</button>
</template>

<script>
  export default {
    name: "MyTest",
    props: {
      setNum:{
        type: Function,
        // 不像对象或数组的默认，这不是一个工厂函数。
        // 这会是一个用来作为默认值的函数
        default() {
          return 'Default function'
        }
      }
    }
  };
</script>
```

Vue 对 props 的书写并没有特别严格的规范要求，但是 Vue 推荐父组件传递的数据时候要使用`-`进行分割连接：

```vue
<template>
  <my-test :set-num="setNum" />
</template>

<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    methods:{
      setNum(){
        console.log("Parent component function is call!")
      }
    }
  }
</script>
```

在字符串模版中（或者 SFC 文件中），传递的属性名`setNum`虽然不会报错，但是 Vue 的组件都是以 Web Components 为标准的，它本身也是符合 XHTML 规范的，所以推荐使用`-`进行分割，而子组件注册属性的时候可以使用驼峰的写法，因为注册的时候使用的是 JS 的逻辑：

```vue
<script>
  export default {
    name: "MyTest",
    props: {
      setNum: Function
    }
  };
</script>
```

另外因为 Vue 单向数据流的限制，每次父组件更新后，所有的子组件中的 props 都会被更新到最新值，这意味着你不应该在子组件中去更改一个 prop。若你这么做了，Vue 会在控制台上向你抛出警告：

```vue
<script>
  export default {
    name: "MyTest",
    props: {
      num: Number
    },
    created() {
      // 错误！prop 是只读的！
      this.num++
    }
  };
</script>
```

如果你想要更改一个 prop 的需求通常来源于以下两种场景：

1. prop 被用于传入初始值；而子组件想在之后将其作为一个局部数据属性。在这种情况下，最好是新定义一个局部数据属性，从 props 上获取初始值即可：

```vue
<script>
  export default {
    name: "MyTest",
    props: {
      num: Number
    },
    data() {
      return {
        superNum: this.num
      }
    };
</script>
```

2. 需要对传入的 prop 值做进一步的转换。在这种情况中，最好是基于该 prop 值定义一个计算属性：

```vue
<script>
  export default {
    name: "MyTest",
    props: {
      num: Number
    },
    computed: {
      // 该 prop 变更时计算属性也会自动更新
      normalizedSize() {
        return this.num * 100;
      }
    }
    </script>
```

## 🔢 props 校验

上面我们说了可以给 props 加上类型校验，如果父组件传递的值不满足类型要求将抛出警告，但是`undefined`和`null`这两个值比较特殊：

```vue
<template>
  <!-- 这将不会产生警告 -->
  <my-test :title="title" :content="content" />
</template>

<script>
  import MyTest from "./components/MyTest.vue";

  export default{
    name:"App",
    components:{
      MyTest
    },
    data(){
      return{
        title: undefined,
        content: null
      }
    }
  }
</script>
```

```vue
<script>
  export default {
    name: "MyTest",
    props:{
      title: String,
      content: String
    }
  }
</script>
```

`undefind`和`null`可以通过任意的数据类型检查，这是因为传递的数据或者是由接口动态返回的，数据到底返回什么是不明确的，但是它本身确实是 String 类型，如果类型检查不通过，这可能会导致程序无法正常运行，但是这并不是程序的 bug，只是程序运行的一种现象！

另外，props 还可以配置某个属性为必传：

```vue
<script>
  export default {
    name: "MyTest",
    props:{
      // 必传，且为 Strign 类型
      title: {
        type:String,
        required: true
      },
    }
  }
</script>
```

props 还可以编写自定义验证函数：

```vue
<script>
  export default {
    name: "MyButton",
    props: {
      btnType: {
        required: true,
        // 自定义验证函数
        validator(value) {
          // 如果不传递 btnType , validator 就不会执行
          console.log(value);
          // 如果值不在数组中将抛出警告
          return ["primary", "danger", "warning", "success"].includes(value);
        }
      }
    }
  };
</script>
```

<br/>warning
⚠️ 注意

`props.*.validator`验证是在当前组件实例创建之前参数的，所以你不能使用 data、computed 中的数据！！！

<br/>
