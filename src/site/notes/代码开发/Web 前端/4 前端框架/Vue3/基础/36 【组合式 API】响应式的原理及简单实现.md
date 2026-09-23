---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/36 【组合式 API】响应式的原理及简单实现/","dg-note-properties":{}}
---

## Vue2 和 3 响应式的区别

首先我们要知道，不是 Vue2 变成了 Vue3，而是 Vue2 的主要思想选项式 API 到 Vue3 版本的时候在选项式 API 的基础上新增了组合式 API，拓展了一些函数的 API，开发者仍然可以选择使用选项 API 进行开发！

函数 API 的好处是可以把组件内部的逻辑进行抽离，不再非得把所有的数据和逻辑都写在组件内部，当该组件内部逻辑复杂的时候会导致文件特别长需要上下翻阅，比较麻烦。当我们使用函数 API 把逻辑抽离到外部的 JS 文件，这样对封装集成更加的友好。

这就是为什么很多人觉得 Vue3 的组合式 API 上手比较困难，是因为我们习惯了 Vue2 的架子模版，在选项式 API 时我们只要在对应的地方编写对应的逻辑即可，组合式 API 显的更加的灵活。

## Vue2 的响应式

首先要知道什么是响应式？

响应式就是数据和视图直接的联动关系。说白了，就是数据更改的时候不需要开发者手动的去操作 DOM，而是让底层的 ViewModel 进行驱动，帮我们追踪数据依赖的变化，并更新视图。

通常，我们是这样定义一个对象：

```js
const obj = {
  a: 1,
  b: 2
};

obj.a = 2;
```

如果我们想要实现，当`a`属性被重新赋值的时候要去 update 更新视图，显然这样是啥都干不了的！

我们希望每次对`a`属性更改的时候都能去 update 视图，所以 Vue2 使用了`Object.defineProperty()`对对象进行了封装，对对象的属性进行拦截。

```js
Object.defineProperty(data, "a", {
  // getter 函数
  get() {
    return data.a;
  },
  // setter 函数
  set(newValue) {
    // update() 更新视图
    data.a = newValue;
  }
});
```

> 每一个对象的属性都具备定义 getter/setter 的权限。
>

如果我们的数据是一个多层嵌套的关系，那么我们就需要封装一个方法然后进行递归：

```js
// 定义我们的数据
const data = {
  a: 1,
  b: {
    c: 2
  },
  d: [1, 2, 3, 4, 5]
};

observe(data);

function observe(data) {
  // 遍历 data 对象
  for (const key in data) {
    defineReactive(data, key, data[key]);
  }
}

function defineReactive(data, key, value) {
  // 如果属性的值是一个对象，那就进行递归
  if ({}.toString.call(value) === "[object Object]") {
    observe(value);
  }

  // 对属性进行拦截操作
  Object.defineProperty(data, key, {
    get() {
      console.log("GET", key);
      return value;
    },
    set(newVal) {
      console.log("SET", key);
      // update() 更新视图
      value = newVal;
    }
  });
}
```

> [!tip]
>
> 本案例只有数据拦截的大概原理，不涉及到`update()`更新视图的逻辑！


![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686292135077-24f1d772-7728-433a-b824-aa9c9ab57e72.png)

打开控制台，你会发现给每一个属性都设置了 getter/setter 的机制。

但是，你会发现当数组调用方法的时候，是无法触发 setter 机制的。这是因为`Object.defineProperty()`方法主要是给对象定义属性的，数组某些方法并不能被劫持，只能通过重新给数组赋值来触发。

如下，使用`push()`方法时候是无法触发 setter 机制的，但是`data.d`的数据确实发生了变化。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686292599551-8e4cb8f3-98e7-4dc1-8184-76017dbd2f3d.png)

如果把`data.d`重新赋值就可以触发 setter 机制。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686292631350-11791fe6-7031-4fe6-a142-0fb9f008f29d.png)

Vue2 想出的解决办法就是把不能触发 setter 机制的方法全部重新实现：

```js
// 定义一个 data 对象
let data = {
  name: 'xiechen',
  age: [1, 2, 3]
}

// 执行观察者模式
observer(data)

// 专门用于劫持数据的
function observer(target) {
  if (typeof target !== 'object' || typeof target == null) {
    return target
  }

  // 如果是数组
  if (Array.isArray(target)) {

    // 保存数组原本的原型
    let oldArrayPrototype = Array.prototype
    // 创建一个空对象，原型指向 oldArrayPrototype
    let proto = Object.create(oldArrayPrototype)

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

这样在数组执行`push()`方法时候，我们可以执行一些其他的操作，最后执行的还是`Array.prototype.push`方法。

以上就是 Vue2 做响应式的大致流程，也就是数据劫持的过程，在劫持的过程中进行数据更新。

## Vue3 的响应式

Vue3 使用了 ES6 种的 Proxy API，因为当时编写 Vue2 的时候 Proyx 的兼容性不是很好所以就放弃使用 Proxy。

Proxy 和`Object.defineProperty()`做的事情很类似，但是又不相同，例如：

```js
function reactive(data) {
  return new Proxy(data, {
    get(target, key) {
      console.log("get", key);

      const value = Reflect.get(target, key);
      // 如果属性的值是一个对象那就进行递归
      return typeof value === "object" ? reactive(value) : value;
    },
    set(target, key, newVal) {
      console.log("set", key);
      // update()

      // 为什么不用 target[key] = newVal?
      // 所有的程序脱离语义化都是败笔，一会 obj.xxx 一会 obj.xx=xx 太乱
      return Reflect.set(target, key, newVal);
    }
  });
}

const data = {
  a: 1,
  b: {
    c: 2
  },
  d: [1, 2, 3, 4, 5]
};

const $data = reactive(data);

console.log($data);
```

以上就是使用 Proxy 对属性进行拦截的大致流程，如果你还不会 Proxy 和 Reflect 对象，那你真的应该抓紧了！

经过 Proxy 的处理会得到一个代理对象：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686293497759-c827b801-2323-40d6-8da8-5578e87c1fd2.png)

Proxy 相比`Object.defineproperty()`的好处：

1、不用逐个属性的定义 getter/setter 函数，默认就是对第一层全部的属性进行劫持，可以进行递归达到深层次拦截的目的。

2、Proxy 的实例对象返回的是针对原对象的一个代理对象

3、可以监听到数组方法的操作

例如对数据进行更改：

```js
const $data = reactive(data);
$data.b.c = 100;
$data.d.push(6);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686293719009-a542cf6f-d0cb-48d5-ac4e-447b6258b554.png)

以上就是 Vue3响应式的大致原理！
