---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/22 keep-alive 的钩子函数与属性/","dg-note-properties":{}}
---

当一个组件在视图中消失的时候会出现什么情况呢？

```vue
<template>
  <div class="tab">
    <div class="nav">
      <div
        v-for="(value, key) in navData"
        :key="key"
        :class="['nav-item', { active: key === currentTab }]"
        @click="setCurrentTab(key)">
        {{ value }}
      </div>
    </div>
    <div class="component">
      <component :is="currentComponent"></component>
    </div>
  </div>
</template>

<script>
  import Intro from "./components/Intro/index.vue";
  import Article from "./components/Article/index.vue"
  import List from "./components/List/index.vue"

  export default {
    components: {
      Intro,
      Article,
      List
    },
    data() {
      return {
        currentTab: "intro",
        navData: {
          intro: "Intro",
          article: "Article",
          list: "List"
        }
      };
    },
    computed: {
      currentComponent() {
        switch (this.currentTab) {
          case "intro":
            return Intro;
          case "article":
            return Article;
          case "list":
            return List;
        }
      }
    },
    methods: {
      // 切换 Tba 的值
      setCurrentTab(key) {
        this.currentTab = key;
      }
    }
  };
</script>
```

```js
export default {
  name: "Intro",
  mounted() {
    console.log("Intro mounted!");
  },
  unmounted() {
    console.log("Intro unmounted!");
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683783708312-aa3d94ec-d295-4256-9a01-f0b62eb4a47c.png)

当我们从 Intero Tab 切换到 Article Tab 的的时候，能够看到 Intro 组件的`mounted`和`unmounted`都进行了触发。

这是因为`<component>`组件动态渲染的时候是不会进行缓存组件的实例，每次切换 Tab 的时候组件都会进行重新初始化！

之前我们说过 Vue 模版渲染的大致流程：

- `createApp`创建应用实例；
- `template`生成 AST 树（AST 树主要描述了`template`是什么样子的，这有点像虚拟节点，AST 树上有很多 Vue 本身的东西，例如 v-if、v-for、v-show、v-on...）；
- 形成 AST 树后才能根据树的内容最后变为 JS 的逻辑，并且过滤掉浏览器不认识的结构；
- 最后 AST 树 => vNode 虚拟节点 => vDom 虚拟元素 => rDom 真实元素；

每当视图要更新又会执行如下的流程：

- 数据变化更新内容 => 产生 vNode => old vNode => compare 进行比较 => diff 寻找差异 => patch 打补丁 => 更新 rDom 描述 => 根据 patch => 更新真实 Dom

回到上面的案例，我们在切换 Tab 的时候，Vue 底层会进行计算，判断 vNode 有没有发生变化：

- 没有：重新组装 vNode => 更新 DOM
- 有：现有的 vNode => 更新 DOM
    - `<keep-alive>`会缓存当前组件的 vNode，不再进行重新初始化渲染

所以我们可以把`<component>`用`<keep-alive>`进行包裹：

```html
<!-- 其他内容 -->
<tempalte>
  <div class="component">
    <keep-alive>
      <component :is="currentComponent"></component>
    </keep-alive>
  </div>
</tempalte>
```

## 钩子函数

因为`<keep-alive>`会缓存组件的实例，不再进行初始化，所以当我们切换 Tab 的时候 Intro 组件的`mounted`和`unmounted`生命周期函数将不会再执行，取而代之的是`<keep-alive>`的钩子函数`activated`和`deactivated`：

```js
export default {
  name: "Intro",
  mounted() {
    console.log("Intro mounted!");
  },
  unmounted() {
    console.log("Intro unmounted!");
  },
  activated() {
    console.log("Intro activated!");
  },
  deactivated() {
    console.log("Intro deactivated!");
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683784763485-74dc925d-f17d-4c92-8bb2-4dadf02cb694.png)

因为 Intro 组件没有进行销毁，所以不会执行`unmounted`函数！

## 属性

`<keep-alive>`允许我们传递 prop：

1、`excludle`表示排除某个组件不进行缓存；

`excludle`属性接收组件的名称，默认是找组件的`name`属性，如果组件内部没有`name`属性则找`components`注册时提供的`key`值！

```html
<!-- 接收字符串，多个组件名称用 , 分割 -->
<keep-alive exclude="Intro,List">
  <component :is="currentComponent"></component>
</keep-alive>
```

```html
<!-- 接收数组 -->
<keep-alive :exclude="['Intro', 'List']">
  <component :is="currentComponent"></component>
</keep-alive>
```

```html
<!-- 接收正则 -->
<keep-alive :exclude="/n|c/">
  <component :is="currentComponent"></component>
</keep-alive>
```

2、`include`表示包含某个组件进行缓存；

`include`和`excludle`属性一致，都按照优先组件的`name`属性，如果组件内部没有`name`属性则找`components`注册时提供的`key`值！

```html
<!-- 接收字符串，多个组件名称用 , 分割 -->
<keep-alive include="Intro,List">
  <component :is="currentComponent"></component>
</keep-alive>
```

```html
<!-- 接收数组 -->
<keep-alive :include="['Intro', 'List']">
  <component :is="currentComponent"></component>
</keep-alive>
```

```html
<!-- 接收正则 -->
<keep-alive :include="/n|c/">
  <component :is="currentComponent"></component>
</keep-alive>
```

> [!warning]
>
> `exclude`和`include`都推荐数组的写法！


3、`max`表示限制可被缓存的最大组件实例数量；

如果缓存达到了 n 个组件，在创建新的组件实例之前，缓存组件时间最久的且没有被访问的组件实例会被销毁！

```html
<keep-alive :max="2">
  <component :is="currentComponent"></component>
</keep-alive>
```

## 模拟简易的 keep-alive

```plain
Demo
│  ├─ index.html
│  ├─ package-lock.json
│  ├─ package.json
│  └─ src
│     ├─ components
│     │  ├─ Comp1.js
│     │  ├─ Comp2.js
│     │  └─ Comp3.js
│     └─ mian.js
```

我们的 Demo 使用 Vite 进行编译启动，package.json 文件内容如下：

```json
{
  "name": "Demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "vite"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "vite": "^4.3.5"
  }
}

```

首先现在 index.html 文件中搭建一个 Tab 的布局：

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
      <!-- Tab 按钮区域 -->
      <div id="buttons">
        <span data-key="comp1">COMP 1</span>
        <span data-key="comp2">COMP 2</span>
        <span data-key="comp3">COMP 3</span>
      </div>
      <div id="wrapper"></div>
    </div>
    <script type="module" src="./src/mian.js"></script>
  </body>
</html>
```

index.html 文件逻辑都放到了 main.js 文件中：

```js
const app = {

}
```

首先，我们需要导入 components 下面的 3 个组件：

```js
import comp1 from "./components/Comp1";
import comp2 from "./components/Comp2";
import comp3 from "./components/Comp3";

const App = {
  components: {
    comp1,
    comp2,
    comp3
  }
};
```

然后我们需要获取页面中三个按钮元素，并绑定点击事件：

```js
import comp1 from "./components/Comp1";
import comp2 from "./components/Comp2";
import comp3 from "./components/Comp3";

const oBtns = document.getElementById("buttons");
const oWrapper = document.getElementById("wrapper");

const App = {
  components: {
    comp1,
    comp2,
    comp3
  },
  bindEvent() {
    oBtns.addEventListener("click", this.handleBtnClick.bind(this), false);
  },
  handleBtnClick(e) {
    const tar = e.target;
    const tagName = tar.tagName.toLowerCase();
  }
};
```

拿到 btn 按钮后，我们要把组件的内容转换为一个简易的 vNode 对象：

```js
import comp1 from "./components/Comp1";
import comp2 from "./components/Comp2";
import comp3 from "./components/Comp3";

const oBtns = document.getElementById("buttons");
const oWrapper = document.getElementById("wrapper");

const App = {
  components: {
    comp1,
    comp2,
    comp3
  },
  bindEvent() {
    oBtns.addEventListener("click", this.handleBtnClick.bind(this), false);
  },
  handleBtnClick(e) {
    const tar = e.target;
    const tagName = tar.tagName.toLowerCase();

    if (tagName === "span") {
      const key = tar.dataset.key;
      const vNode = this.setVNode(this.components[key]); // 传递组件对象
      const rNode = this.setRNode(vNode); // 传递 vNode 对象

      oWrapper.innerHTML = "";
      oWrapper.appendChild(rNode); // 渲染 rNode
    }
  },
  setVNode(comp) {
    const { template, name } = comp;
    const regTag = template.match(/\<(.+?)\>/)[1]; // 获取标签的名称
    const regContent = template.match(/\>(.+?)\</)[1]; // 获取元素的内容

    // 例如 {tag:'h1', children:'This is COMP-1', mark:'Comp1'}
    return {
      tag: regTag,
      children: regContent,
      mark: name
    };
  },
  setRNode(vNode) {
    const { tag, children, mark } = vNode;

    // 根据 tag 创建对应的 DOM 元素
    const node = document.createElement(tag);
    node.innerText = children;

    // 然后创建后的 DOM
    return node;
  }
};
```

最后我们还需要执行一下我们整个 App 对象：

```js
import comp1 from "./components/Comp1";
import comp2 from "./components/Comp2";
import comp3 from "./components/Comp3";

const oBtns = document.getElementById("buttons");
const oWrapper = document.getElementById("wrapper");

const App = {
  components: {
    comp1,
    comp2,
    comp3
  },
  init() {
    this.bindEvent();
  },
  bindEvent() {
    oBtns.addEventListener("click", this.handleBtnClick.bind(this), false);
  },
  handleBtnClick(e) {
    const tar = e.target;
    const tagName = tar.tagName.toLowerCase();

    if (tagName === "span") {
      const key = tar.dataset.key;
      const vNode = this.setVNode(this.components[key]); // 传递组件对象
      const rNode = this.setRNode(vNode); // 传递 vNode 对象

      oWrapper.innerHTML = "";
      oWrapper.appendChild(rNode); // 渲染 rNode
    }
  },
  setVNode(comp) {
    const { template, name } = comp;
    const regTag = template.match(/\<(.+?)\>/)[1]; // 获取标签的名称
    const regContent = template.match(/\>(.+?)\</)[1]; // 获取元素的内容

    // 例如 {tag:'h1', children:'This is COMP-1', mark:'Comp1'}
    return {
      tag: regTag,
      children: regContent,
      mark: name
    };
  },
  setRNode(vNode) {
    const { tag, children, mark } = vNode;

    // 根据 tag 创建对应的 DOM 元素
    const node = document.createElement(tag);
    node.innerText = children;

    // 然后创建后的 DOM
    return node;
  }
};

App.init();
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683790831411-2dfadb25-13a0-4612-bca6-953b379d2399.gif)

到这里我们就简单了实现了一个 Tab 的切换，下面我要给`<div id="wrapper"></div>`加上`<keep-alive>`进行包裹，进行缓存。

```html
<body>
  <div id="app">
    <div id="buttons">
      <span data-key="comp1">COMP 1</span>
      <span data-key="comp2">COMP 2</span>
      <span data-key="comp3">COMP 3</span>
    </div>
    <keep-alive>
      <div id="wrapper"></div>
    </keep-alive>
  </div>
  <script type="module" src="./src/mian.js"></script>
</body>
```

```js
import comp1 from "./components/Comp1";
import comp2 from "./components/Comp2";
import comp3 from "./components/Comp3";

const oBtns = document.getElementById("buttons");
const oWrapper = document.getElementById("wrapper");

const App = {
  components: {
    comp1,
    comp2,
    comp3
  },
  compCache: {}, // 新增一个缓存对象
  init() {
    this.bindEvent();
  },
  bindEvent() {
    oBtns.addEventListener("click", this.handleBtnClick.bind(this), false);
  },
  handleBtnClick(e) {
    const tar = e.target;
    const tagName = tar.tagName.toLowerCase();

    /* if (tagName === "span") {
      const key = tar.dataset.key;
      const vNode = this.setVNode(this.components[key]); // 传递组件对象
      const rNode = this.setRNode(vNode); // 传递 vNode 对象

      oWrapper.innerHTML = "";
      oWrapper.appendChild(rNode); // 渲染 rNode
    } */

    // 这里 vNode 将不再直接调用 setVNode 返回，而是要进行判断
    if (tagName === "span") {
      const key = tar.dataset.key;
      let vNode = null;

      // 如果缓存对象中存在 key
      if (this.compCache[key]) {
        vNode = this.compCache[key];
      }else {
        // 否则直接去调用 setVNode 生成 vNode
        vNode = this.setVNode(this.components[key]);

        // 如果 oWrapper 的父节点是 <keep-alive> 那就把当前组件的 vNode 存到 compCache 中去
        if (this.checkKeppAlive(oWrapper)) {
          this.compCache[key] = vNode;
        }
      }
    }

    // 将 vNode 转换为 rNode ，最后进行渲染
    const rNode = this.setRNode(vNode);
    oWrapper.innerHTML = "";
    oWrapper.appendChild(rNode);

  },
  setVNode(comp) {
    const { template, name } = comp;
    const regTag = template.match(/\<(.+?)\>/)[1]; // 获取标签的名称
    const regContent = template.match(/\>(.+?)\</)[1]; // 获取元素的内容

    // 例如 {tag:'h1', children:'This is COMP-1', mark:'Comp1'}
    return {
      tag: regTag,
      children: regContent,
      mark: name
    };
  },
  setRNode(vNode) {
    const { tag, children, mark } = vNode;

    // 根据 tag 创建对应的 DOM 元素
    const node = document.createElement(tag);
    node.innerText = children;

    // 然后创建后的 DOM
    return node;
  },
  checkKeppAlive(wrapper) {
    const outerWrapper = wrapper.parentNode.tagName.toLowerCase();
    return outerWrapper === "keep-alive";
  }
};

App.init();
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683791394428-fb0dfba1-4a90-4c03-8c87-ca08abe85b2e.gif)

我们还可以添加两个钩子函数，当组件激活的时候执行：

```js
import comp1 from "./components/Comp1";
import comp2 from "./components/Comp2";
import comp3 from "./components/Comp3";

const oBtns = document.getElementById("buttons");
const oWrapper = document.getElementById("wrapper");

const App = {
  components: {
    comp1,
    comp2,
    comp3
  },
  mounted(callback) {
    callback && callback();
  },
  activated(callback) {
    callback && callback();
  },
  compCache: {}, // 新增一个缓存对象
  init() {
    this.bindEvent();
  },
  bindEvent() {
    oBtns.addEventListener("click", this.handleBtnClick.bind(this), false);
  },
  handleBtnClick(e) {
    const tar = e.target;
    const tagName = tar.tagName.toLowerCase();

    /* if (tagName === "span") {
      const key = tar.dataset.key;
      const vNode = this.setVNode(this.components[key]); // 传递组件对象
      const rNode = this.setRNode(vNode); // 传递 vNode 对象

      oWrapper.innerHTML = "";
      oWrapper.appendChild(rNode); // 渲染 rNode
    } */

    // 这里 vNode 将不再直接调用 setVNode 返回，而是要进行判断
    if (tagName === "span") {
      const key = tar.dataset.key;
      let vNode = null;

      // 如果缓存对象中存在 key
      if (this.compCache[key]) {
        vNode = this.compCache[key];
      }else {
        // 否则直接去调用 setVNode 生成 vNode
        vNode = this.setVNode(this.components[key]);

        // 如果 oWrapper 的父节点是 <keep-alive> 那就把当前组件的 vNode 存到 compCache 中去
        if (this.checkKeppAlive(oWrapper)) {
          this.compCache[key] = vNode;
        }
      }
    }

    // 将 vNode 转换为 rNode ，最后进行渲染
    const rNode = this.setRNode(vNode);
    oWrapper.innerHTML = "";
    oWrapper.appendChild(rNode);

    if (this.checkKeppAlive(oWrapper)) {
      this.activated(() => {
        console.log(key, "activated");
      });
    } else {
      this.mounted(() => {
        console.log(key, "mounted");
      });
    }
  },
  setVNode(comp) {
    const { template, name } = comp;
    const regTag = template.match(/\<(.+?)\>/)[1]; // 获取标签的名称
    const regContent = template.match(/\>(.+?)\</)[1]; // 获取元素的内容

    // 例如 {tag:'h1', children:'This is COMP-1', mark:'Comp1'}
    return {
      tag: regTag,
      children: regContent,
      mark: name
    };
  },
  setRNode(vNode) {
    const { tag, children, mark } = vNode;

    // 根据 tag 创建对应的 DOM 元素
    const node = document.createElement(tag);
    node.innerText = children;

    // 然后创建后的 DOM
    return node;
  },
  checkKeppAlive(wrapper) {
    const outerWrapper = wrapper.parentNode.tagName.toLowerCase();
    return outerWrapper === "keep-alive";
  }
};

App.init();
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1683791630401-6e430a0c-9889-4515-803c-c3a7e025b3a0.gif)

源码地址：

[JSPlusPlus/腾讯课堂/Vue本尊07/01 at main · xiechen1201/JSPlusPlus](https://github.com/xiechen1201/JSPlusPlus/tree/main/%E8%85%BE%E8%AE%AF%E8%AF%BE%E5%A0%82/Vue%E6%9C%AC%E5%B0%8A07/01)
