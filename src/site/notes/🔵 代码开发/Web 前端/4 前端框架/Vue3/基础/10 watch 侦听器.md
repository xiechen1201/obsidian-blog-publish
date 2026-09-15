---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/基础/10 watch 侦听器/","dg-note-properties":{}}
---

## 🔢 基本认识

`watch`和`computed`的区别：

1、`computed`计算属性，关注点在模版，主要是抽离和复用模版中复杂的数据逻辑。

特点：当函数内部的依赖发生变化的后，数据才会重新计算。

2、`watch`侦听器，关注点在数据（`data`或`computed`中的数据）更新，主要负责给数据增加监听，当数据更新的时候监听器函数就会执行。

特点：数据更新的时候，需要完成什么样的逻辑。

```js
export default {
  data(){
    return{
      result: 0
    }
  },
  watch: {
    // 可以获取到数据更新的新值和旧值
    result(newVal, oldVal) {
      console.log(newVal, oldVal);
    }
  }
};
```

## 🔢 简单实现

例如我们实现一个简单的`watch`侦听器，先看一下我们的结构目录：

```markdown
├─ 03-watch
  ├─ Vue.js
  ├─ index.html
  ├─ main.js
  ├─ reactive.js
  └─ watcher.js
```

首先我们先去`main.js`文件中写出我们大致的结构：

```js
import Vue from "./Vue.js";

const vm = new Vue({
  data() {
    return {
      result: 0
    };
  },
  watch: {
    // 监听 data 中的 result
    result(newVal, oldVal) {
      console.log("watch result:", newVal, oldVal);
    }
  }
});

console.log(vm);

vm.result = 100;
vm.result = 200;

console.log(vm.result);
```

接下来我们专注`Vue.js`文件就可以了，我们采用`ES6`的类来书写：

```js
class Vue {
  constructor(options) {
    // 这里是 Vue 类的入口
  }

  init(vm, watch) {
    // 这里主要处理一些初始化的数据
  }

  initData(vm) {
    // 处理 data() 中的响应式数据
  }

  initWatcher(vm, watch) {
    // 处理 watch 中的监听数据
  }
}

export default Vue;
```

以上就是我们今天`Vue`文件的大概结构，下面我们先来处理`data`中的数据：

```js
import { reactive } from "./reactive.js";

class Vue {
  constructor(options) {
    // 我们在实例 Vue 的时候，传递进来一个对象，所以我们可以进行解构
    const { data, watch } = options;

    // 因为 data 中的数据是可以直接被访问到的，就像这样 vm.result
    // 所以我们需要把 data 中的数据挂在到 Vue 实例上面
    this.$data = data();
    this.init(this, watch);
  }

  init(vm, watch) {
    // 调用初始化 data
    this.initData(vm);
  }

  initData(vm) {
    // 我们把处理 data 这部分的代码，抽离出去为一个 reactive.js 文件
    // reactive 方法介绍 3 个参数：当前实例、当 get data 中数据的回调、当 set data 中数据的回调
    reactive(vm, (key, value) => {}, (key, newVal, oldVal) => {});
  }
}

export default Vue;
```

处理`data`中的数据：

```js
export function reactive(vm, __get__, __set__) {
  const _data = vm.$data;

  // 遍历 data 中的数据，也就是 { result: 0 }
  for (const key in _data) {
    // 添加拦截
    Object.defineProperty(vm, key, {
      get() {
        // 当获取 vm.result 的时候就会执行回调，且返回数据
        __get__(key, _data[key]);
        return _data[key];
      },
      set(newVal) {
        // 当给 vm.result 设置值的时候就会执行回调，且赋值
        const oldVal = _data[key];

        _data[key] = newVal;
        __set__(key, newVal, oldVal);
      }
    });
  }
}
```

到这里我们已经处理好`data`的数据了，实例数据如下：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675670037322-71200e8e-4a25-496d-82f3-cdb94f60784e.png)

根据`reactive.js`的`set`方法，我们这样打算：既然到`result`改变的时候，我们可以拦截到，且执行了回调方法，那我们在回调了去执行`watch`不就可以了吗？

那么我们就继续来写`watcher.js`文件：

```js
class Watcher {
  constructor() {
    this.watchers = [];
    /
     * watchers 的数据结构：
     * 	{
     *  	key: result
     *  	fn: result(newVal, oldVal)
     * 	}
     *
     * 通过 addWatcher(vm, watcher, key) 去往 watchers 里面添加数据
     *  */
  }

  // 往 watchers 里面添加数据
  addWatcher(vm, watcher, key) {
    this._addWatchProp({
      key,
      fn: watcher[key].bind(vm)
    });
  }

  invoke(key, newVal, oldVal) {
    // 用 addWatcher 保存的值去遍历对比
    this.watchers.map((item) => {
      if (item.key === key) {
        // 调用 result() 传入新值、旧值
        item.fn(newVal, oldVal);
      }
    });
  }

  _addWatchProp(watchProp) {
    this.watchers.push(watchProp);
  }
}

export default Watcher;
```

接着我们在`Vue.js`文件中引入`watcher.js`：

```js
import { reactive } from "./reactive.js";
import Watcher from "./watcher.js";

class Vue {
  constructor(options) {
    // 我们在实例 Vue 的时候，传递进来一个对象，所以我们可以进行解构
    const { data, watch } = options;

    // 因为 data 中的数据是可以直接被访问到的，就像这样 vm.result
    // 所以我们需要把 data 中的数据挂在到 Vue 实例上面
    this.$data = data();
    this.init(this, watch);
  }

  init(vm, watch) {
    // 调用初始化 data
    this.initData(vm);

    // 初始化 watch，且传入实例对象和 watch 对象（也就是 { watch: { result:xxx }}）
    // 然后我们就能得到一个 watcher 的实例，并把它保存到实例对象上面，方面后面直接调用执行
    const watcherIns = this.initWatcher(vm, watch);
    this.$watch = watcherIns.invoke.bind(watcherIns);
  }

  initData(vm) {
    // 我们把处理 data 这部分的代码，抽离出去为一个 reactive.js 文件
    // reactive 方法介绍 3 个参数：当前实例、当 get data 中数据的回调、当 set data 中数据的回调
    reactive(vm, (key, value) => {}, (key, newVal, oldVal) => {
      // 当设置 vm.result 的时候我们就去执行 watch 的 invoke 方法
      this.$watch(key, newVal, oldVal);
    });
  }

  initWatcher(vm, watch) {
    // 枚举 watch 然后新增监听器
    // 返回实例，实例有调用 watch 的方法，执行监听器
    const watcherIns = new Watcher();

    for (const key in watch) {
      // 实例、watch 对象、result
      watcherIns.addWatcher(vm, watch, key);
    }

    return watcherIns;
  }
}

export default Vue;
```

![watchers 的结果](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675670988447-0d613203-4f4f-4bd9-9f45-36083f5e867e.png)

这样就算完成了，当我们去改变`vm.result`的时候，`watch`中的`result()`方法就会被执行。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675671500413-db15a93d-16b3-4d55-abde-58c3f0c6fff5.png)

源码地址：

[https://github.com/xiechen1201/JSPlusPlus/blob/main/%E8%85%BE%E8%AE%AF%E8%AF%BE%E5%A0%82/Vue%E6%9C%AC%E5%B0%8A03/03/main.js](https://github.com/xiechen1201/JSPlusPlus/blob/main/%E8%85%BE%E8%AE%AF%E8%AF%BE%E5%A0%82/Vue%E6%9C%AC%E5%B0%8A03/03/main.js)
