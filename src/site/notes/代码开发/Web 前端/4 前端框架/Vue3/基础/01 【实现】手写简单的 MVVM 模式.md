---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/01 【实现】手写简单的 MVVM 模式/","dg-note-properties":{}}
---

上一篇文章我们写了`MVC`模式，以及说了`MVC`模式的缺点，这篇文章我们就来实现一个简单的`MVVM`模式。

`Model`负责管理数据，`View`负责管理视图，`ViewModel`负责数据和视图的连接。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675739226623-1bd4ab24-98d0-467e-9909-fc18b1f32a0f.png)

先看一下大概的目录结构：

```markdown
08-MVVM
 ├─ index.html
 ├─ mvvm
 │  ├─ index.js
 │  ├─ render.js # 负责页面的渲染
 │  ├─ compiler
 │  │  ├─ event.js # 负责事件处理
 │  │  └─ state.js # 负责数据处理
 │  ├─ reactive
 │  │  ├─ index.js
 │  │  └─ mutableHandler.js # 对数据进行响应式处理
 │  └─ shared
 │     └─ utils.js
 └─ src
    └─ App.js # 程序的入口
```

这个案例使用了`Vite`作为服务器进行开发，所以你需要进行安装`Vite`和配置`package.json`文件。

```js
{
  "name": "08-mvvm",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "vite"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "vite": "^4.0.3"
  }
}
```

## 🔢 App.js

```js
// Vite 会自动补全 /index.js 的后缀
import { useDom, useReactive } from "../mvvm/index";

function App() {
  // 创建响应式
  const state = useReactive({
    count: 0,
    name: "TestName"
  });
  // 操作 state 数据的方法
  const add = function (num) {
    state.count += num;
  };
  const minus = function (num) {
    state.count -= num;
  };
  const changeName = function (name) {
    state.name = name;
  };
  // 模版的渲染
  return {
    template: `
      <h1>{{ count }}</h1>
      <h2>{{ name }}</h2>
      <button onClick="add(2)">新增</button>
      <button onClick="minus(1)">减去</button>
      <button onClick="changeName('xiechen')">更改名字</button>
    `,
    state,
    methods: {
      add,
      minus,
      changeName
    }
  };
}

useDom(
  App(), // 返回 template，state，methods
  document.querySelector("#app")
);
```

以上代码，我们引入了`useDom`方法对`App`的模版进行渲染，引入`useReactive`对`state`数据进行管理。

我们把所有需要用到的方法，都导入到了`mvvm/index.js`这个文件里面进行管理：

```js
export { useReactive } from "./reactive";
export { useDom, update } from "./render";
export { eventFormat } from "./compiler/event";
export { stateFormat } from "./compiler/state";
```

接下来，就让我们看看每个文件都负责干了点啥。

## 🔢 mvvm/reactive

该文件的`useReactive`方法在`App.js`文件中进行了调用：

```js
// isObject 主要是判断是不是一个对象，如果你想看到更多的实现细节，你可以滑倒文章的最后。
import { isObject } from "../shared/utils";
import { mutableHandler } from "./mutableHandler";

export function useReactive(target) {
  // target 为 App.js 中的 state , 也就是
  /*
    {
      count: 0,
      name: "TestName"
    }
  */
  // mutableHandler 为 Proxy 对象拦截属性的一些方法
  return createReactObject(target, mutableHandler);
}

function createReactObject(target, baseHandler) {
  if (!isObject(target)) {
    return target;
  }
  return new Proxy(target, baseHandler);
}
```

以上代码，我们在`createReactObject`方法中对`App.js`文件中的数据进行拦截，并引入了`mutableHandler.js`文件对`Proxy`的数据进行处理。

```js
import { useReactive } from "./index";
/
 * hasOwnProperty 用于判断一个属性是不是对象本身上的属性，而非原型上的属性
 * isEqual 用于判断新值和旧值是否相等
 */
import { isObject, hasOwnProperty, isEqual } from "../shared/utils";
import { update } from "../render";
import { statePool } from "../compiler/state";

function createGetter() {
  return function get(target, key, receiver) {

    // 通过 Reflect.get 方法去操作属性
    // 详见MDN：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/get
    const res = Reflect.get(target, key, receiver);

    // 如果返回的值是一个对象，那么就继续调用 useReactive 去处理
    if (isObject(res)) {
      return useReactive(res);
    }

    // 否则直接返回
    return res;
  };
}

function createSetter() {
  return function set(target, key, value, receiver) {
    const isKeyExist = hasOwnProperty(target, key);
    const oldValue = target[key];
    // 同 Reflect.get ，但是 set 返回的是是否设置成功的布尔值
    const res = Reflect.set(target, key, value, receiver);

    // 如果对象上没有这个属性，那么这个属性就是新增的属性
    if (!isKeyExist) {
      console.log("响应式新增：", value);
    } else if (!isEqual(value, oldValue)) {
      // 否则就是去更改属性的值
      console.log("响应式修改：", key, value);
      // 然后调用视图的 update 方法
      update(statePool, key, value);
    }

    return res;
  };
}

const get = createGetter();
const set = createSetter();

export const mutableHandler = {
  get,
  set
};
```

以上代码，我们分别对属性的`set`和`get`拦截进行了处理，在`set`方法中，无论是新增或更改对象的属性，我们都可以拦截的到。

## 🔢 mvvm/render.js

`render.js`文件主要负责了对视图管理，我们在`App.js`文件中调用了`useDom`方法进行视图的渲染，且在`Proxy`的`set`处理中调用了`update`进行视图的更新。

```js
/
 * bindEvent 用于给元素绑定事件
 */
import { bindEvent } from "./compiler/event";
import { eventFormat, stateFormat } from "./index";

export function useDom({ template, state, methods }, rootDom) {
  // 接收 App.js 方法返回的对象，也就是
  /*
    {
      template: xxx
      state: xxx
      methods: xxx
    }
  */
  // 调用 render 方法，对模版数据进行处理
  rootDom.innerHTML = render(template, state);
  // 调用 bindEvent 方法进行绑定事件
  bindEvent(methods);
}

export function render(template, state) {
  /*
    eventFormat 方法会给绑定事件的模版新增一个属性 data-mark="xxx"，例如模版

    <button onClick="add(2)">新增</button> =>>
    <button data-mark="12345" onClick="add(2)">新增</button>

    并且保存到一个名为 eventPool 的数组中，数据结构如下：
    [
      {
        mark: 12345 // dom 标签上的 data-mark
        handler: add(2)
        type: click
      }
    ]
  */
  template = eventFormat(template);

  /*
    stateFormat 方法会给标签新增一个属性 data-mark="xxx"，例如模版
    <h1>{{ count }}</h1> =>>
    <h1 data-mark="12345">1</h1>

    并且保存到一个名为 statePool 的数组中，数据结构如下：
    [
      {
         mark: 12345,
         state: ["count"]
       }
    ]
  */
  template = stateFormat(template, state);
  return template;
}

/*
   update 方法接收了 statePool 为参数，也就是
   [
        {
          mark: 12345
          state: ["count"]
        }
      ]

    还接收了 set 数据的时候，要更改的属性和值
*/
export function update(statePool, key, value) {
  const allElements = document.querySelectorAll("*");
  let oItem = null;

  // 进行遍历
  statePool.forEach((el) => {
    // 如果 statePool 中 el.state 中的数据等于要 set 的属性名
    if (el.state[el.state.length - 1] === key) {

      for (let i = 0; i < allElements.length; i++) {
        oItem = allElements[i];
        const _mark = parseInt(oItem.dataset.mark);

        // 如果 statePool.mark 等于某个节点的 data-mark 属性
        if (el.mark === _mark) {
          oItem.innerHTML = value;
        }
      }
    }
  });
}
```

以上代码，我们分别调用了

- `bindEvent`对`DOM`绑定事件。
- `eventFormat`把`DOM`和事件的对应关系进行存储。
- `stateFormat`把`DOM`和数据的对应关系进行存储，并且替换为`state`中对应的数据。

## 🔢 mvvm/compiler

以下是对`mvvm/compiler/event.js`文件的详解：

```js
import { checkType, randomNum } from "../shared/utils";

/
 * {
 *  mark: random,
 *  handler: 事件处理函数的字符串
 *  type: click
 * }
 */
const reg_onClick = /onClick\=\"(.+?)\"/g;
const reg_fnName = /^(.+?)\(/;
const reg_arg = /\((.*?)\)/;
const eventPool = [];

export function eventFormat(template) {
  return template.replace(reg_onClick, function (node, key) {
    const _mark = randomNum();

    // 把数据的对应关系存到 eventPool 里面，方面我们进行对比调用
    eventPool.push({
      mark: _mark,
      handler: key.trim(),
      type: "click",
    });

    /*
      eventPool 结构如下：
      [
        {
          mark: 12345 // dom 标签上的 data-mark
          handler: add(2)
          type: click
        }
      ]
    */

    // 给标签新增一个 data-mark="12345" 这样的属性
    return `data-mark="${_mark}"`;
  });
}

export function bindEvent(methods) {
  const allElements = document.querySelectorAll("*");
  let oItem = null;
  let _mark = 0;

  /*
    eventPool 结构如下：
    [
      {
        mark: 12345 // dom 标签上的 data-mark
        handler: add(2)
        type: click
      }
    ]
  */

  // 循环对比
  eventPool.forEach((el) => {
    for (let i = 0; i < allElements.length; i++) {
      oItem = allElements[i];
      _mark = parseInt(oItem.getAttribute("data-mark"));

      // 如果 eventPool 中 el.mark 等于某个 dom 的 data-mark 的属性
      if (el.mark === _mark) {
        // 绑定事件
        oItem.addEventListener(el.type, function () {
          const fnName = el.handler.match(reg_fnName)[1];
          const arg = checkType(el.handler.match(reg_arg)[1]);

          // 调用 state.methods 里面对应的方法
          methods[fnName](arg);
        }, false);
      }
    }
  });
}
```

以下是对`mvvm/compiler/state.js`文件的详解：

```js
import { randomNum } from "../shared/utils";

const reg_html = /\<.+?\>\{\{(.+?)\}\}\<\/.+?\>/g;
const reg_tag = /\<(.+?)\>/;
const reg_var = /\{\{(.+?)\}\}/g;

/
 * {
 *  mark: _mark
 *  state: value
 * }
 */

export const statePool = [];

let o = 0;

export function stateFormat(template, state) {
  let _state = {};

  // 绑定 data-mark
  template = template.replace(reg_html, function (node, key) {
    const matched = node.match(reg_tag);
    const _mark = randomNum();

    /*
      _state 结构如下：
      {
        mark: 12345,
      }

      statePool 结构如下：
      [
        {
          mark: 12345,
        }
      ]
    */

    _state.mark = _mark;
    statePool.push(_state);

    _state = {};

    // 例如将 <h1>{{ count }}</h1> 替换为
    // <h1 data-mark="12345">{{ count }}</h1>
    return `<${matched[1]} data-mark="${_mark}">{{ ${key} }}</${matched[1]}>`;
  });

  // 替换模版数据
  template = template.replace(reg_var, function (node, key) {
    let _var = key.trim(); // 拿到 state 里面属性的 key
    const _varArr = _var.split(".");
    let i = 0;

    while (i < _varArr.length) {
      // 去拿 state 里面对应的数据，例如 _var 为 count，所以 state.count
      // 最后 _var 得到了 state.count 的值，也就是 0
      _var = state[_varArr[i]];
      i++;
    }

    _state.state = _varArr;
    statePool[o].state = _varArr;
    o++;

    /*
      statePool 的结构如下：
      [
        {
          mark: 12345
          state: ["count"]
        }
      ]
    */

    // 将 <h1 data-mark="12345">{{ count }}</h1> 中的 count 替换为真实的数据
    return _var;
  });

  return template;
}
```

## 🔢 shared/utils.js

`utils.js`文件主要存放的是一些工具类的方法，我们在上面案例使用到的方法，在这里都可以找得到。

```js
function isObject(val) {
  return typeof val === "object" && val !== null;
}

function hasOwnProperty(target, key) {
  return Object.prototype.hasOwnProperty.call(target, key);
}

function isEqual(newVal, oldValue) {
  return newVal === oldValue;
}

function randomNum() {
  return new Date().getTime() + parseInt(Math.random() * 10000);
}

function checkType(str) {
  const reg_check_str = /^[\'\"](.*?)[\'\"]/;
  const reg_str = /(\'|\")/g;

  if (reg_check_str.test(str)) {
    return str.replace(reg_str, "");
  }

  switch (str) {
    case "true":
      return true;
    case "false":
      return false;
    default:
      break;
  }

  return Number(str);
}

export { isObject, hasOwnProperty, isEqual, randomNum, checkType };
```

## 🔢 收尾

到这里，我们就把这个模式完整的解释完了，再回顾一下这个代码的结构，

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675753833730-83d45332-01ba-4ebf-be08-22b774a87b64.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675753905750-6af96387-35c4-449d-b4fb-2279a3c1efa7.gif)

这样我们就只负责数据和视图的逻辑，剩下的事情全部交给`mvvm`驱动去管理，`mvvm`负责了创建响应式数据、对事件和数据进行编译、对模版进行渲染以及视图更新。

源码地址：

[JSPlusPlus/腾讯课堂/Vue本尊01/08-MVVM/src/App.js at main · xiechen1201/JSPlusPlus](https://github.com/xiechen1201/JSPlusPlus/blob/main/%E8%85%BE%E8%AE%AF%E8%AF%BE%E5%A0%82/Vue%E6%9C%AC%E5%B0%8A01/08-MVVM/src/App.js)
