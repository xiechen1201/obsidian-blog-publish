---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/34 【组合式 API】provide 与 inject/","dg-note-properties":{}}
---

我们之前学习选项式 API 的时候已经写过一篇文章：

[19、依赖注入 provide/inject](https://www.yuque.com/xiechen/de54wv/wuiuyqdscrgrgr9x)

## 🔢 基础使用

本篇主要看一下组合式 API provide/inject 的区别，在组合式 API 中 provide/inject 都需要从 Vue 中进行导入：

```js
import { provide } from 'vue';

export default {
  setup(){
    provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
  }
}
```

`provide()`函数接收两个参数。第一个参数被称为注入名，可以是一个字符串或是一个 Symbol。后代组件会用注入名来查找注入的值。

第二个参数是提供的值，值可以是任意类型，包括响应式的状态，比如一个 ref：

```js
import { provide, ref } from 'vue';

export default {
  setup(){
    const count = ref(0);
    provide('key', count);
  }
}
```

同样的后代组件如果想要使用注入的值，需要从 Vue 中导入`inject()`方法：

```js
import { inject } from 'vue'

export default {
  setup() {
    const message = inject('message');
    return { message }
  }
}
```

默认情况下，如果该注入名没有任何组件提供，则会抛出一个运行时警告。

如果在注入一个值时不要求必须有提供者，那么我们应该声明一个默认值，和 props 类似：

```js
import { inject } from 'vue'

export default {
  setup() {
    // 如果没有祖先组件提供 "message"
    // `value` 会是 "这是默认值"
    const value = inject('message', '这是默认值');
    return { value }
  }
}
```

## 🔢 和响应式数据配合使用

当提供/注入响应式的数据时，要尽量的把变更数据的方法都写在父组件内，而不是让子组件去更改提供的数据，使其更容易维护！！！

如果子组件想要更改提供的数据，我们要在供给方组件内声明并提供一个更改数据的方法函数：

```js
// 在供给方组件内
import { provide, ref } from 'vue'

export default {
  setup(){
    const location = ref('North Pole')
    function updateLocation() {
      location.value = 'South Pole'
    }

    provide('location', {
      location,
      updateLocation
    })
  }
}
```

```js
// 在注入方组件内
import { inject } from 'vue'

export default {
  setup(){
    const { location, updateLocation } = inject('location');
    // 执行方法去更改供给方的数据
    updateLocation();
  }
}
```

如果你想确保提供的数据不能被注入方的组件更改，你可以使用`readonly()`来包装提供的值。

```js
import { ref, provide, readonly } from 'vue'

export default {
  setup(){
    const count = ref(0);
    // 用 readonly() 把数据设置为只读
    provide('read-only-count', readonly(count));
  }
}
```
