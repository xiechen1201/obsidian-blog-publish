---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/20 依赖注入 provide inject/","dg-note-properties":{}}
---

provide 意为「提供」，inject 意为「注入」，它们主要通过 provide 在组件内部提供一个子组件能够访问的数据，然后子组件通过 inject 在内部注入数据。

我们都知道我们在开发 Vue 项目的时候，都是以视图为线索的，把视图分割为无数的碎片（也就是组件，下文都会说是碎片）。然后把无数的碎片组合成我们想要的样子和功能。

碎片和碎片之间相互嵌套就会产生依赖关系，最终组合成一个页面交互的整体，这个整体就是组件树。

被嵌套的组件数据来源可能源于父组件，这样组件就需要通过 prop 进行定义配置。

原则上，组件是可以无限制的被依赖，数据也可以无限制的被传递下去，这就是形成了正常的单向数据流。

但是无限制的嵌套关系就会导致数据将穿过所有依赖关系中的组件，也就造成了许多层组件并未使用的数据出现。如果想要解决这个问题，就应该尽量让组件的依赖关系变得简单，组件之间的嵌套关系不能太深，组件化设计的时候就要考虑到组件的扁平化！！！

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683278031580-e0f49d5f-7030-4b54-a081-318a924f4299.png)

但是某些复杂的项目，我们又不得不嵌套的非常深，这个时候 provide/inject 就登场啦！

简单说就是父组件通过 provide 提供数据，子组件通过 inject 注入数据。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683278053273-f8247707-a0ff-4298-868d-02bac333ffbd.png)

但是 provide/inject 又存在着很大的弊端：

1、父组件 provide 一个数据，无论哪个层级的子组件通过 inject 注入数据，这个数据都不是响应式的；

2、父组件不知道哪个子组件使用了 provide 的数据，子组件也不会知道谁提供了 inject 的数据；

## 🔢 使用案例

那么 provide/inject 如何做呢？

1、父组件 provide 提供一个数据

```js
export default {
  provide: {
    message: 'hello!'
  }
}
```

假如我们不想直接把`message`的内容定义好，而是要获取`data`中的某个属性，那么`provide`就不能是一个对象！！！而是改用函数的方式，这是因为`data`的数据是可变的，因为对象赋值是通过引用赋值的，当`data`中的数据发送变化时`provide`也会跟着发送变化，这样就会影响子组件的依赖数据！

```js
export default {
  data() {
    return {
      message: 'hello!'
    }
  },
  provide: {
    // ❌❌❌
    // 这将会导致报错，因为无法获取到 this
    message: this.message
  }
}
```

```js
export default {
  data() {
    return {
      message: 'hello!'
    }
  },
  provide() {
    // ✅✅✅
    // 使用函数的形式，可以访问到 `this`
    return {
      message: this.message
    }
  }
}
```

2、子组件通过 inject 来注入父组件提供的数据

```js
export default {
  inject: ['message'],
  created() {
    console.log(this.message) // "hello"
  }
}
```

而这里的`inject`数据时可以直接在`data`中拿到的，例如：

```js
export default {
  inject: ['message'],
  data() {
    return {
      // 基于注入值的初始数据
      fullMessage: this.message
    }
  }
}
```

默认情况下，`inject`注入名会被某个父组件提供。但如果该注入名的确没有任何组件提供，则会抛出一个运行时警告。

如果在注入一个值时不要求必须有提供者，那么我们应该声明一个默认值，和`props`类似：

```js
export default {
  // 当声明注入的默认值时
  // 必须使用对象形式
  inject: {
    message: {
      from: 'message', // 当与原注入名同名时，这个属性是可选的
      default: 'default value'
    },
    user: {
      // 对于非基础类型数据，如果创建开销比较大，或是需要确保每个组件实例
      // 需要独立数据的，请使用工厂函数
      default: () => ({ name: 'John' })
    }
  }
}
```

## 🔢 配合响应式

上面我们说了，`provide`提供的数据默认是不会响应式的，如果你想要实现响应式，你需要使用`computed()`函数提供一个计算属性：

```js
export default {
  data() {
    return {
      message: 'hello!'
    }
  },
  provide() {
    return {
      // 显式提供一个计算属性
      message: Vue.computed(() => this.message)
    }
  }
}
```

```js
import { computed } from 'vue'

export default {
  data() {
    return {
      message: 'hello!'
    }
  },
  provide() {
    return {
      // 显式提供一个计算属性
      message: computed(() => this.message)
    }
  }
}
```
