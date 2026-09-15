---
{"dg-publish":true,"permalink":"//web/4/vue3//12-v-for/","dg-note-properties":{}}
---

`v-for`可以用来循环数据，基本语法是`v-for="指令表达式"`，例如循环一个数组：

```js
const app = {
  template: `
  <li v-for="(item, index) in items">
    {{ item.message }}
  </li>`,
  data(){
    return {
      items: [{ message: 'Foo' }, { message: 'Bar' }]
    }
  }
}
```

在使用`v-for`指令的时候，`index`属性表示当前项的下标，它是可选的。

你也可以使用`of`作为分隔符来替代`in`，这更接近`JavaScript`的迭代器语法：

```js
const app = {
  template: `
  <li v-for="(item, index) of items">
    {{ item.message }}
  </li>
  <li v-for="(value, key, index) in person">
    {{ value }}
  </li>
  `,
  data(){
    return {
      items: [{ message: 'Foo' }, { message: 'Bar' }],
      person: {
        name: "张三",
        age: 28
      }
    }
  }
}
```

建议遍历可迭代对象时使用`(item, index) of array`，枚举对象的时候使用`(value, key, index) in object`

<br/>

## 🔢 遍历数组

```html
<ul>
  <li v-for="(item, index) of list" :key="item.id">{{ item.name }}</li>
</ul>
```

建议在使用`v-for`的时候搭配`key`属性，`key`属性必须是唯一的，方便`Vue`进行「就地更新策略」。

`key`值不建议使用数组的下标，这是因为当我们删除/新增项的时候不能保证`key`绝对的不变化；如果你的列表不会进行新增/删除数组的时候，可以使用`index`作为`key`。

## 🔢 枚举对象

```js
const app = {
  template: `
  <ul>
      <li v-for="(value, key, index) in privateInfo">
      	{{ key }}: {{ value }}
    		<template v-if="key === 'hobbies'">
      		<span v-for="(item, index) of value"> {{ item }}、 </span>
      	</template>
      </li>
  </ul>
  `,
  data(){
    return {
      privateInfo: {
        name: "Crystal",
        age: 18,
        hobbies: ["Travel", "Piano"]
      }
    }
  }
}
```

当使用`v-for`枚举对象的时候，遍历的顺序会基于对该对象调用`Object.keys()`的返回值来决定。

## 🔢 范围值

`v-for`可以直接接受一个整数值。在这种用例中，会将该模板基于 1...n 的取值范围重复多次。

```html
<span v-for="n in 10">{{ n }}</span>
```

需要注意的是，`n`的初值是从 1 开始而非 0。

## 🔢 组件上使用 v-for

`v-for`可以直接应用在组件上使用，和在一般的元素上使用没有区别，同样需要提供`key`属性：

```js
const app = {
  template: `
    <MyComponent
      v-for="(item, index) in items"
      :index="index"
      :key="item.id"
  	/>`
}
```

但是，这不会自动将任何数据传递给组件，因为组件有自己独立的作用域。这会使组件与`v-for`的工作方式紧密耦合。明确其数据的来源可以使组件在其他情况下重用。

如果想将迭代后的数据传递到组件中，我们还需要传递`props`。

```js
const myComponent = {
  props: ['item']
}

const app = {
  template: `
    <MyComponent
      v-for="(item, index) in items"
      :item="item"
      :index="index"
      :key="item.id"
  	/>`
}
```

## 🔢 数组变化侦测

在`Vue3`中，`Vue`能够侦听响应式数组的变更方法，并在它们被调用时触发相关的更新。这些变更方法包括：

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

但是在`Vue2`在，`Vue`通过重写数组相关的方法来监听数组的变更，通过在数组的原型上新增了一层原型来达到拦截：

![Vue2 中数组的原型](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1676258067867-4182b7b8-5311-458f-b9da-834559caaf21.jpeg)

这是因为`Object.defineProperty()`只能实现对对象的属性进行拦截，无法对数组的属性进行拦截，例如下面的例子：

```js
var vm = {
  data: {
    a: 1,
    b: 2,
    list: [1, 2, 3, 4, 5]
  }
};

for (const key in vm.data) {
  Object.defineProperty(vm, key, {
    get() {
      console.log("数据获取");
      return vm.data[key];
    },
    set(newVal) {
      console.log("数据设置");
      vm.data[key] = newVal;
    }
  });
}
```

以上代码，我们定义了`data`，`data`里有数组，然后我们分别去操作这些属性看看变化。

当我们直接把数组赋值为一个新数组的时候，数组是可以被拦截的：

```js
vm.a = 1;
vm.list = [2, 3, 4, 5, 6];
console.log(vm.list);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1676258382075-ab18e8fc-0718-4226-a8dd-28766616b39c.png)

当我们调用`push`方法的时候，数据确实发生了变化，但是`set`机制却没有触发：

```js
vm.list.push(6);
console.log(vm.list);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1676258465124-145e5522-621a-4c46-9f82-220503850549.png)

可以看到图片中，出现两次“数据获取”，这是因为我们在第一行中`vm.list`的时候执行了`get`机制，然后才能调用`push()`方法。

`Object.defineProperty()`没办法监听下列方法对数组的变更：

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

但是当数组重新赋值的是却能被拦截的到，所以`Vue2`对上面这些方法进行了包裹封装（类似重写），大致的响应式原理如下：

```js
// 定义一个 data 对象
// Object.defineProperty 可以重新定义属性 给属性安插 getter setter 方法
let data = {
  name: 'xiechen',
  age: [1, 2, 3]
}

// Array.prototype.push = function() {}
data.age.push(4)

// 执行观察者模式
observer(data)

// 专门用于劫持数据的
function observer(target) {
  if (typeof target !== 'object' || typeof target == null) {
    return target
  }

  if (Array.isArray(target)) {

    // 保存数组原本的原型
    let oldArrayPrototype = Array.prototype
    let proto = Object.create(oldArrayPrototype) // 继承

    Array.from(['push', 'shift', 'unshift', 'pop']).forEach(method => {
      // 函数劫持，把函数重写
      proto[method] = function () {
        // 执行数组原本的方法
        oldArrayPrototype[method].call(this, ...arguments)
        // 更新视图
        updateView()
      }
    })

    // 给数组新增一个原型，target.__proto__ = proto
    Object.setPrototypeOf(target, proto)
  }

  // 如果是对象直接执行响应式
  for (let key in target) {
    defineReactive(target, key, target[key])
  }
}

// 执行响应式
function defineReactive(target, key, value) {
  // 递归执行
  observer(value)
  // Object.defineProperty 只能劫持对象
  Object.defineProperty(target, key, {
    get() {
      return value
    },
    set(newVal) {
      if (newVal !== value) {
        value = newVal
        updateView()
      }
    }
  })
}

function updateView() {
  console.log('更新视图');
}
```

以上就是`Vue2`对对象和数组进行的响应式拦截大概思路。

那么我们替换原数组的数据是否会重新渲染整个`DOM`列表（性能原因）？

不一定，`Vue`在对`dom`操作的时候进行了大量的新旧节点信息的对比算法，`Vue`会把`dom`重新渲染的程度最小化，做到已有的`dom`节点最大化的复用。

## 🔢 v-for 和 v-if 联合使用

`Vue`不推荐在同一个元素上使用`v-if`和`v-for`指令。

```html
<!-- 不推荐 -->
<ul>
  <li
    v-for="(item, index) of todoList"
    v-if="!item.completed"
    :key="item.id">
    	{{ item.content }}
  </li>
</ul>
```

以上代码，我们在`li`元素上使用了`v-for`指令进行循环`todoList`数组进行渲染，使用`v-if`指令判断`completed`为`false`时才会显示，然后你就会发现一个错误！

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1676612564955-1ce04dd2-4ebf-4188-a277-68d6bb7247ec.png)

错误的意思说：`item`属性在渲染期间确实被访问了，但是`item`并没有定义。

这是因为`Vue3`在进行`v-for`解析的时候，`v-if`的优先级是高于`v-for` 的，这就会导致`v-if`获取不到`item`。

<br/>

那么如何解决这个问题呢？

1、我们可以利用`template`进行包裹，让`v-for`循环`template`标签

```html
<ul>
	<!--  这样 v-for 的优先级就会比 v-if 高  -->
  <template v-for="(item, index) of todoList" >
    <li v-if="!item.completed" :key="item.id">
      {{ item.content }}
    </li>
  </template>
</ul>
```

2、使用`computed`过滤数组

```js
const app = {
  template: `
    <ul>
      <li v-for="(item, index) of notCompletedTodoList" :key="item.id">
        {{ item.content }}
      </li>
    </ul>
  `,
  data(){
    return{
      todoList: [
        // ...
      ]
    }
  },
  computed:{
    notCompletedTodoList(){
      return this.todoList.filter(el=> !el.completed)
    }
  }
}
```

但是像这样的情况例外，你可以在一个元素上同时使用`v-for`和`v-if`指令：

```html
<ul>
  <li v-if="todoList.length > 0" v-for="(item, index) of todoList" :key="item.id">
    {{ item.content }}
  </li>
</ul>
```

这是因为`v-if`并不依赖`v-for`解构的属性，所以你可以进行判断！

在`Vue2`中，`v-for`的优先级是高于`v-if`的，到了`Vue3`版本的时候把它们的优先级进行了调整互换，从下面两方面来看这是必然的：

1、逻辑层：`v-if`是优先级要大于`v-for`的，`v-if`决定了是否要进行渲染，`v-for`决定了如何进行渲染。

2、性能层：如果先使用`v-for`判断如何进行渲染，再使用`v-if`判断是否渲染在性能上所带来的消耗是不一样的。
