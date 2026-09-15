---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/基础/09 computed 计算属性/","dg-note-properties":{}}
---

`computed`用于解决模版中复杂的逻辑运算，或者逻辑需要被复用的地方。

```js
// bad
const App = {
  template: `
    <h1>{{ studentCount > 0 ? "学生数量："+ studentCount : "暂无学生" }}</h1>
    <h1>{{ studentCount > 0 ? "学生数量："+ studentCount : "暂无学生" }}</h1>
  `,
  data() {
    return {
      studentCount: 1
    };
  },
};
```

以上代码的问题有两个方面：

1、模版的逻辑和样式要尽可能的绝对分离，模版上不要做太多的逻辑。

2、如果另一个地方要显示同样的逻辑，这样就会导致需要运算两次。

<br/>

## 🔢 计算属性的特点

1、多次复用相同值的数据，计算属性只会调用一次

```js
const App = {
  template: `
    <h1>{{ studentCountInfo }}</h1>
    <h1>{{ studentCountInfo }}</h1>
  `,
  data() {
    return {
      studentCount: 1
    };
  },
  computed: {
    studentCountInfo() {
      // Invoked 只会被执行一次
      console.log("Invoked");
      return this.studentCount > 0 ? "学生数量：" + this.studentCount : "暂无学生";
    }
  }
};

const vm = createApp(App).mount("#app");
```

2、计算属性只会在内部逻辑依赖的数据发生变化的时候才会重新调用。

```js
const App = {
  template: `
    <h1>{{ studentCountInfo }}</h1>
    <h1>{{ studentCountInfo }}</h1>
    <button @click="clickBtn">按钮</button>
    <button @click="clickBtn2">按钮</button>
  `,
  data() {
    return {
      studentCount: 1
    };
  },
  computed: {
    studentCountInfo() {
      console.log("Invoked");
      return this.studentCount > 0 ? "学生数量：" + this.studentCount : "暂无学生";
    }
  },
  methods: {
    clickBtn() {
      this.studentCount = new Date();
    },
    clickBtn2() {
      this.studentCount = 2;
    }
  }
};
```

![只有数据变化后，才会打印 Invoked](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673936181197-1d433ada-d775-4ae4-ae20-78cb6ecc9be2.gif)

3、计算属性会缓存其依赖的上一次计算出的数据结果

## 🔢 computed 和 methods 的区别

1. 在使用时，`computed`可以当做属性使用，而`methods`则可以当做方法调用。
2. `computed`可以具有`getter`和`setter`方法，因此可以赋值，而`methods`是不行的。

```js
const App = {
  // ...
  computed: {
    studentCountInfo() {
      console.log("Invoked");
      return this.studentCount > 0 ? "学生数量：" + this.studentCount : "暂无学生";
    },
    result: {
      get() {},
      set() {}
    }
  }
}
```

3. `computed`无法接收多个参数，而`methods`可以。
4. `computed`是有缓存的，而`methods`没有。
5. `computed`属性会暴露到应用/组件的实例上。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673937128552-1a697041-cc5b-4b13-ae7d-f84a9616c252.png)
