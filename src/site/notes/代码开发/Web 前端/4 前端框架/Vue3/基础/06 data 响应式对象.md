---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/06 data 响应式对象/","dg-note-properties":{}}
---

## 🔢 认识 data 选项

在`Vue`中，`data`必须是一个函数：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="app"></div>
		<!-- 使用 Vue3 的 CDN -->
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="./main.js"></script>
  </body>
</html>
```

```js
const { createApp } = window.Vue;

const app = Vue.createApp({
  template: `
      <h1>{{ title }}</h1>
    `,
  // 将 data 赋值为一个对象
  data:{
    title: "this is title"
  }
});

const vm = app.mount("#app");
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673256519420-0dfbf781-79b0-4023-bdfd-27a78e366f07.png)

这是因为`Vue`在创建实例的过程中会「执行」`data`函数，然后返回数据对象。并通过响应式进行包装`data`存储到「实例对象」的`$data`属性中。

```js
const { createApp } = window.Vue;

const app = Vue.createApp({
  template: `
      <h1>{{ title }}</h1>
    `,
  data(){
    return {
      title: "this is title"
    }
  }
});

const vm = app.mount("#app");
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1674981236522-7fe7f078-88d4-48ee-bfeb-2782a11ebb36.png)

我们可以通过实例直接访问到`data`中的数据，而不需要访问`$data`。

```js
const vm = app.mount("#app");

console.log(vm.title); // "this is title"
console.log(vm.$data.title); // "this is title"
```

这是因为`vm.title`和`vm.$data.title`指向的是同一个数据引用，如果你直接给实例对象新增一个属性的话，这个属性并不会新增到`$data`对象中，`$data`对象是响应式数据只在初始化的时候对数据进行定义拦截。

```js
const { createApp } = window.Vue;

const app = Vue.createApp({
  template: `
      <h1>{{ title }}</h1>
    `,
  data() {
    return {
      title: "this is title",
    };
  },
});
const vm = app.mount("#app");

// 新增一个 author 属性
vm.author = "xiechen";

console.log(vm);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673257155131-dd80466e-693d-4012-8424-66c94f4ee72c.png)

在`vm`实例化对象中，以`$`、`_`开头的属性都是`Vue`内置的属性或`api`，开发者应尽量的避免用这些前缀命名自己的变量或方法！！！

<br/>

## 🔢 模拟 Vue 把 data 数据挂载到实例上

实现的思路其实也非常的简单，上面我们说了`vm.title`和`vm.$data.title`是同一个数据的引用，所以我们按照这个思路来实现。

创建一个`Vue`的构造函数，这样才能返回一个实例化对象。

```js
function VueTest(options){
  // ...
}

// 进行实例化
let vm = new VueTest({
  data(){
    return {
      a: 1,
      b: 2
    }
  }
})
```

然后我们专心写构造函数内部的逻辑。

```js
function VueTest(options){
  // 因为 data 是个函数，所以我们需要执行后才能得到数据
  // 需要把 $data 挂载到实例对象上
  this.$data = options.data();
}

let vm = new VueTest({
  data(){
    return {
      a: 1,
      b: 2
    }
  }
})
console.log(vm);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673257676927-99db7c8b-4d24-4d7c-a45c-0a9d528603a4.png)

既然`$data`已经挂载到实例上了，下面我们还需要把`data`中的属性也挂载到实例上。

```js
function VueTest(options){
  this.$data = options.data();

  for (const key in this.$data) {

    // 把 key 都定义到 this 对象上，也就是当前实例对象
    Object.defineProperty(this, key, {
      get: function () {
        return this.$data[key];
      },
      set: function (newValue) {
        this.$data[key] = newValue;
      }
    })

  }
};
```

我们利用`Object.defineProperty`对属性进行拦截，当访问`vm.a`的时候实际上访问的是`vm.$data.a`。

最后来测试一下：

```js
function VueTest(options) {
  this.$data = options.data();

  for (const key in this.$data) {
    Object.defineProperty(this, key, {
      get: function () {
        return this.$data[key];
      },
      set: function (newValue) {
        this.$data[key] = newValue;
      },
    });
  }
}

var vm = new VueTest({
  data() {
    return {
      a: 1,
      b: 2,
    };
  },
});

console.log(vm.a);
vm.b = 3;
console.log(vm);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673258084198-8219f7cf-8cc8-48b6-bb1a-9ba8405e28f1.png)

可以看到`vm`可以直接访问`a`属性，`a`属性会被拦截，实际访问的仍然是`$data.a`。

## 🔢 data 为什么必须要是一个函数？

这是一个老生常谈的问题了，大家都知道`JS`中的对象是引用类型，如果把一个对象赋值给另外一个对象，则新对象赋值的其实是原对象的堆内存地址。

```js
var obj1 = { a:1 };
var obj2 = obj1;
obj2.b = 2;

console.log(obj1); // {a: 1, b: 2}
console.log(obj2); // {a: 1, b: 2}
```

如果`data`是一个对象就会出现上面的问题，当同时实例化两个`Vue`应用（或者在`Vue`页面上多次使用同一个组件）就会造成以上的引用数据的问题。

```js
function VueTest(options) {
  // 不用执行 data ,而是直接赋值
  this.$data = options.data;

  for (const key in this.$data) {

    Object.defineProperty(this, key, {
      get: function () {
        return this.$data[key];
      },
      set: function (newValue) {
        this.$data[key] = newValue;
      },
    });

  }
}
```

```js
var data = {
  a: 1,
  b: 2,
};

var vm1 = new VueTest({
  data: data,
});
var vm2 = new VueTest({
  data: data,
});

vm1.b = 3;

console.log(vm1);
console.log(vm2);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1673271647479-c1bcc4c1-3422-4ac2-91ef-bbd58d23e801.png)

`Vue`中的`data`之所以要求是一个函数，就是为了确保每次执行的时候能返回一个新的对象，确保每个实例数据的引用都是独一无二的。

<br/>
