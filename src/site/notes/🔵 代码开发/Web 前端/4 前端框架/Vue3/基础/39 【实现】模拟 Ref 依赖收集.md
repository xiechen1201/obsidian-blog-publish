---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/基础/39 【实现】模拟 Ref 依赖收集/","dg-note-properties":{}}
---

现在有这么一道题目：

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
    <div id="app">
      <h1>{{ title }}</h1>
      <h2>{{ conten }}</h2>
      <h1>{{ title }}</h1>
      <h2>{{ content }}</h2>
      <button @click="setTitle">设置标题</button>
      <button @click="setContent">设置内容</button>
      <button @click="reset">重 置</button>
    </div>
    <script type="module" src="./index.js"></script>
  </body>
</html>
```

```js
import { createApp, ref } from "./vue/index.js";

createApp("#app", {
  refs: {
    title: ref("This is title."),
    content: ref("This is content.")
  },
  methods: {
    setTitle() {
      this.title.value = "这是标题。";
    },
    setContent() {
      this.content.value = "这是内容。";
    },
    reset() {
      this.title.$reset();
      this.content.$reset();
    }
  }
});
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686645378079-0b9409b9-d3fa-43d4-b8ab-b0328973e212.png)

现在的需求是让我们自行去实现`createApp()`和`ref()`方法，来实现 HTML 文件 DOM 中数据的解析和对数据的操作。

首先我们要进行题目的分析：

1、`createApp()`中定义的数据类似 Vue 的选项 API 方式。

2、`createApp()`和`ref()`又类似 Vue 的组合式 API。

3、如何使用`ref()`把数据进行包装，并可以调用方法？

4、如何进行渲染页面？

可以先获取到`#app`节点下所有的 DOM ==> 分析`{{ xxx }}`的内容 ==> 找到`refs`种对应的数据 ==> 更新节点内容

5、当执行方法更改`refs`里面的数据时如何进行更新呢？

对数据进行劫持 ==> 触发 setter 机制 ==> 找到对应的 DOM ==> 更新节点内容

6、如何给元素绑定事件呢？

找到所有的节点 ==> 分析带有`@click`的属性 ==> 绑定事件为`methods`里面的方法

综上，这是我们对这道题目的一个初始的思路，下面我们先一步步的去实现！

1、实现`createApp()`函数

根据题目我们可以看到`createApp()`接收两个参数：根节点、数据对象。

```js
import { ref } from "./hooks.js";

// 接收：根节点、初始化 data
export function createApp(el, { refs, methods }) {
  const $el = document.querySelector(el);
  const allNodes = $el.querySelectorAll("*");

  console.log(allNodes);
}

export { ref };
```

`vue/index.js`作为我们程序的入口文件，自然需要暴露出去一个`createApp()`方法。

但是我们又不想把 Ref 相关的逻辑放在该文件内部，所以我们需要把 Ref 相关的内容也导入进来然后进行导出。

当`createApp()`方法执行的时候，我们可以进行跟节点`#app`然后获取到该节点下所有的 DOM。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686646239659-9ae63637-ba9c-4cf1-a932-54e8e036080c.png)

2、创建 Ref 对象

当`createApp()`函数执行的是，定义的`refs`里数据又会执行`ref()`函数。

那么该如何实现`ref()`函数呢？

1. `ref()`函数返回的数据可以被调用`$reset()`方法，所以`ref()`函数应该返回一个对象。
2. 我们如何知道页面中的那些元素使用了该 Ref 对象呢？可以给该对象设置一个`deps`属性专门依赖的元素。
3. 因为我们需要调用`$reset()`方法来还原数据的初始值，需要还需要设置一个`_defaultValue`来保持默认值。
4. 当我们对 Ref 对象的值进行更改的时候还需要触发劫持来进行更新，所以我们还需要定义`_value`和`value`来表示对象的值，`value`是负责劫持的数据。
5. 因为`refs`里面有多个数据，还有调用方法，我们可以使用实例化构造函数的方式来创建 Ref 对象。

```js
import Ref from "./Ref.js";

export function ref(initialValue) {
  return new Ref(initialValue);
}
```

我们可以把 Ref 类抽离出去，然后导入进行实例化，只传递一个初始化的值就可以了。

```js
// Ref 类
export default class Ref {
  constructor(initialValue) {
    this.deps = new Set(); // 方便我们后面存储 DOM 依赖
    this._defaultValue = initialValue; // 该属性不可变的
    this._value = initialValue; // 该属性可变的
  }

  /*
    为什么不能直接使用 _value ?
    因为我要触发劫持，如果直接操作就无法触发劫持了
  */
  get value() {
    return this._value;
  }

  set value(newValue) {
    console.log("劫持被触发！")
    this._value = newValue;
  }

  $reset() {
    // 重置的时候直接把 _defaultValue 赋值给 _value 就可以了
    this._value = this._defaultValue;
  }
}
```

然后我们就得到了两个 Ref 对象。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686647322492-2c802a0d-cb95-4d56-8342-734d3eaaf1e2.png)

3、收集 Ref 的`deps`依赖

我们现在拿到了所有的节点和所有的 Ref 对象，那我们就可以给 Ref 对象设置`deps`来管理 Ref 和  DOM 的依赖。

```js
import { ref, createRefMap } from "./hooks.js";

export function createApp(el, { refs, methods }) {
  const $el = document.querySelector(el);
  const allNodes = $el.querySelectorAll("*");
  const refsMap = createRefMap(allNodes, refs);

  console.log(refsMap)
}

export { ref };
```

```js
import Ref from "./Ref.js";

// 该正则表达式可以匹配到 {{ xxx }}
const reg_var = /\{\{(.+?)\}\}/;

export function ref(initialValue) {
  return new Ref(initialValue);
}

export function createRefMap(allNodes, refs) {
  // 遍历 allNodes
  allNodes.forEach((element) => {
    // 如果 DOM 的文本内容可以被匹配
    if (reg_var.test(element.innerText)) {
      console.log(element.innerText.match(reg_var));
    }
  });
}
```

这样我们就匹配到了 DOM 中绑定 Ref 的内容。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686647742568-f73f728d-b492-4b70-b64e-a07693694943.png)

最后我们只需要取数组的第 1 位并把对应的 Ref 更改即可。

```js
import Ref from "./Ref.js";

// 该正则表达式可以匹配到 {{ xxx }}
const reg_var = /\{\{(.+?)\}\}/;

export function ref(initialValue) {
  return new Ref(initialValue);
}

export function createRefMap(allNodes, refs) {
  // 遍历 allNodes
  allNodes.forEach((element) => {
    // 如果 DOM 的文本内容可以被匹配
    if (reg_var.test(element.innerText)) {
      // 得到  ['{{title}}', 'title']
      const refKey = element.innerText.match(reg_var)[1].trim();
      refs[refKey].deps.add(element);
    }
  });

  return refs;
}
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686647974641-73558d91-f349-4cea-b4aa-b4b7138fa1ba.png)

这样我们就把对应的 DOM 收集了起来。

4、渲染页面

现在我们有了 Ref 对象，Ref 对象保存了对应的依赖 DOM，那我们就可以去渲染页面啦。

```js
import { ref, createRefMap } from "./hooks.js";
import { render } from "./render.js";

export function createApp(el, { refs, methods }) {
  const $el = document.querySelector(el);
  const allNodes = $el.querySelectorAll("*");
  const refsMap = createRefMap(allNodes, refs);

  // render() 函数只要一个 Ref 对象作为参数即可
  render(refsMap);
}

export { ref };
```

```js
export function render(refs) {
  // 遍历 refs
  // refs 就是
  // {
  //   title: { deps:... value:xxx }
  //   content: { deps:... value:xxx }
  // }

  for (const key in refs) {
    _render(refs[key]);
  }
}

// 我们把 deps 渲染单独抽离为一个函数，因为 update 的时候也需要
function _render({ deps, value }) {
  deps.forEach((element) => {
    element.innerText = value;
  });
}
```

5、绑定事件

接下来就是给按钮绑定事件了。

```js
import { ref, createRefMap } from "./hooks.js";
import { render } from "./render.js";
import { bindEvent } from "./event.js";

export function createApp(el, { refs, methods }) {
  const $el = document.querySelector(el);
  const allNodes = $el.querySelectorAll("*");
  const refsMap = createRefMap(allNodes, refs);

  // render() 函数只要一个 Ref 对象作为参数即可
  render(refsMap);
  // 通过 apply() 来改变 this 指向 Refs
  bindEvent.apply(refsMap, [methods, allNodes]);
}

export { ref };
```

```js
export function bindEvent(methods, allNodes) {
  // 遍历节点
  allNodes.forEach((element) => {
    // 获取含有 @click 属性的值
    const handlerName = element.getAttribute("@click");

    if (handlerName) {
      // 找到 methods 里对应的方法
      element.addEventListener("click", methods[handlerName].bind(this), false);
    }
  });
}
```

到这里，我们点击按钮就会触发对应的事件，然后就会触发 Ref 劫持。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686648962954-27aab1f0-3788-4dbb-8471-3a89a6399cae.gif)

6、更新视图

已经实现了数据的更改，但是视图咋没变化呢？因为我们还没执行`update()`

```js
import { update } from "./render.js";

export default class Ref {
  constructor(initialValue) {
    this.deps = new Set();
    this._defaultValue = initialValue; // 不可变的
    this._value = initialValue; // 可变的
  }

  /*
    为什么不能直接使用 _value ?
    因为我要触发劫持，如果直接操作就无法触发劫持了
  */
  get value() {
    return this._value;
  }

  set value(newValue) {
    console.log("劫持被触发！");
    this._value = newValue;
    // 数据更改后重新渲染数据
    update(this);
  }

  $reset() {
    this._value = this._defaultValue;
    update(this);
  }
}
```

```js
export function render(refs) {
  for (const key in refs) {
    _render(refs[key]);
  }
}

export function update(ref) {
  // 直接调用 _render() 即可
  _render(ref);
}

// 我们把 deps 渲染单独抽离为一个函数，因为 update 的时候也需要
function _render({ deps, value }) {
  deps.forEach((element) => {
    element.innerText = value;
  });
}
```

这样我们就实现了整个程序的运行：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686649231223-3f8d772c-387d-42ae-9df1-e62eb61c6500.gif)

最后，源码地址献上：

[JSPlusPlus/腾讯课堂/Vue本尊10/10/index.js at main · xiechen1201/JSPlusPlus](https://github.com/xiechen1201/JSPlusPlus/blob/main/%E8%85%BE%E8%AE%AF%E8%AF%BE%E5%A0%82/Vue%E6%9C%AC%E5%B0%8A10/10/index.js)
