---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/25 emit 事件/","dg-note-properties":{}}
---

## 触发与监听事件

在组件的模板表达式中，可以直接使用`$emit`方法触发自定义事件：

```html
<!-- MyComponent -->
<button @click="$emit('someEvent')">click me</button>
```

`$emit`事件的时候需要传递一个「自定义事件名称」，事件名推荐使用驼峰 camelCase 的形式。

`$emit()`方法在组件实例上也同样以`this.$emit()`的形式可用：

```js
export default {
  methods: {
    submit() {
      this.$emit('someEvent')
    }
  }
}
```

父组件通过`v-on`或者简写`@`进行监听子组件 emit 来的自定义事件：

```html
<MyComponent @some-event="callback" />
```

父组件在监听事件的时候推荐使用 kebab-case 的方式，因为这符合 HTML 的规范！

## 事件参数

子组件在通过`$emit`一个自定义事件的时候，可以向父组件传递参数：

```html
<button @click="$emit('increaseBy', 1)">
  Increase by 1
</button>
```

然后父组件可以接收到子组件传递来的参数：

```html
<MyButton @increase-by="increaseCount" />
```

```js
export default{
  methods: {
    increaseCount(n) {
      this.count += n
    }
  }
}
```

如果子组件想传递多个参数，我们可以在事件名后面依次传递：

```html
<!-- 子组件 emit 事件 -->
<button @click="$emit('increaseBy', 1, 2, 3, 4, 5)">
  Increase by 1
</button>
```

```js
// 父组件接收
export default{
  methods: {
    increaseCount(...args) {
      console.log(args); // [1, 2, 3, 4, 5]
    }
  }
}
```

但是你应该避免这么操作，而是使用对象去传递多个参数，这会让你的代码更加好维护。

```html
<!-- 子组件 emit 事件 -->
<button @click="$emit('increaseBy', {num1: 1, num2: 2, num3:3 })">
  Increase by 1
</button>
```

父组件可以更好的接收：

```js
// 父组件接收
export default{
  methods: {
    increaseCount({ num1, num2, num3 }) {
      console.log(num1, num2, num3); // 1, 2, 3
    }
  }
}
```

## 声明触发的事件

Vue3 新增了`emits`属性来注册需要 emit 出去的事件：

```js
export default {
  emits: ['inFocus', 'submit'],
  methods:{
    onFocus(){
      this.$emit('inFocus');
    },
    onSubmit(){
      this.$emit('submit');
    }
  }
}
```

这是为了更好的记录组件的 emit 流程，而不再需要去代码文件里翻找！

如果子组件想要 emit 一个原生的事件，例如下面这个示例：

```html
<template>
  <div>
    <!-- 监听子组件传递来的原生事件 -->
    <my-counter @click="getCount" />
  </div>
</template>

<script>
  import MyCounter from "./components/MyCounter.vue";

  export default {
    name: "App",
    components: {
      MyCounter
    },
    methods: {
      // 打印子组件传递来的值
      getCount(count) {
        console.log(count);
      }
    }
  };
</script>
```

```vue
<template>
  <div>
    <h1>{{ count }}</h1>
    <my-button btn-type="primary" @click="setCount">+</my-button>
  </div>
</template>

<script>
  import MyButton from "./MyButton.vue";

  export default {
    name: "MyCounter",
    components: {
      MyButton
    },
    data() {
      return {
        count: 0
      };
    },
    methods: {
      setCount() {
        // $emit 出一个 click 事件
        this.count++;
        this.$emit("click", this.count);
      }
    }
  };
</script>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1684311054244-aef2a783-5242-4f3b-bb9d-5acddd154dc2.gif)

结果你会发现，无论是点击页面中`<button>`元素还是点击`<h1>`元素，都会打印出事件对象`event`。

你可能会有一个疑问？代码中明明打印的是`this.$emit("click", this.count);`出来的`count`属性啊，为啥会打印出`event`对象呢？

这其实和我们前面说过的「透传 attrs」有关系，我们调用子组件的时候绑定了`@click`事件，但是子组件没有注册这个原生的事件，就导致了 click 事件绑定到了`<MyCounter>`组件的根元素上，所以无论你点击页面中的`<button>`元素还是点击`<h1>`元素都产生了「事件冒泡」，父组件直接监听到了所以就会打印出`event`对象！！！

如果要想解决这个问题，你需要在子组件内部通过`emits`注册一下你要`$emit`出去的原生事件，这样 Vue 就能够更好地将事件和透传 attribute 作出区分，从而避免一些由第三方代码触发的自定义 DOM 事件所导致的边界情况。

```js
export default {
  name: "MyCounter",
  components: {
    MyButton
  },
  emits: ["click"],
  data() {
    return {
      count: 0
    };
  },
  methods: {
    setCount() {
      this.count++;
      this.$emit("click", this.count);
    }
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1684311555425-11e7a4e9-247b-4463-91af-9d5181ee3352.gif)

<br/>warning
⚠️ 注意

如果你是 Vue2 的版本，同样的问题，你需要通过修饰符`.navtive`去解决，例如：`<my-counter @click.navtive="getCount" />`

通过`.navtive`来描述代替原生事件！！！

<br/>

## 事件校验

和对 props 添加类型校验的方式类似，所有触发的事件也可以使用对象形式来描述。

要为事件添加校验，那么事件可以被赋值为一个函数，接受的参数就是抛出事件时传入`this.$emit`的内容，返回一个布尔值来表明事件是否合法。

```js
export default {
  name: "MyForm",
  //  emits: ["submit"],
  emits: {
    submit: ({ username, password }) => {
      if (username && password) {
        console.log("Emit successfully!!!");
        return true;
      }
      console.log("Faild to emit!!!")
      return false;
    }
  },
  data() {
    return {
      username: "",
      password: ""
    };
  },
  methods: {
    submitMyForm(e) {
      e.preventDefault();
      this.$emit("submit", {
        username: this.username,
        password: this.password
      });
    }
  }
};
```

以上案例，当`username && password`不通过的时候，父组件仍然能接收到事件和事件参数，只是控制台会进行报警告而已。
