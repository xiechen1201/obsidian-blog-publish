---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/基础/00 【实现】手写简单的 MVC 模式/","dg-note-properties":{}}
---

## 🔢 认识 MVC

`MVC`的概念是从后端开发引入的，全名是`Model View Controller`，是模型(`model`)－视图(`view`)－控制器(`controller`)的缩写，是一种代码的设计模式。

- `Model`：数据模型层，对数据进行增删改查的操作，操作数据库。
- `View`：视图层，显示视图或者视图模版。
- `Controller`：控制器层，该层主要将数据和视图进行关联挂载，和基本的逻辑操作。

它们 3 个的关系大概如下：

前端`View`发起请求 =>> `Controller`接收到请求 =>> 让`Model`去操作数据库 =>> 返回给`Controller` =>> 返回到前端

> 如果后端有`view`层那这就是服务端渲染，由后端渲染完成后返回到前端。
>

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675735145869-59244ac5-6d50-424e-a759-4136995d751e.png)

## 🔢 前端中的 MVC

到了前端中，`MVC`的设计模式和后端基本一致：

- `Model`管理视图需要的数据，数据和视图进行关联。
- `View`视图层，`HTML`模版和视图渲染。
- `Controller`管理事件的逻辑操作。

例如我们实现一个简单的`MVC`结构，实现一个加减乘除的案例。

我们先把整体的分层结构写出来：

```js
(function () {
  function init() {
    model.init(); // 组织数据，监听数据（数据代理）
    view.render(); // 组织 HTML 模版 + 渲染 HTML 模版
    controller.init(); // 事件处理函数的定义和绑定
  }

  // 数据层
  var model = {
  };

  // 视图层
  var view = {
  };

  // 控制层
  var controller = {
  };

  init();
})();
```

然后我们先把`Model`层的数据定义好：

```js
(function () {
  function init() {
    model.init(); // 组织数据，监听数据（数据代理）
    view.render(); // 组织 HTML 模版 + 渲染 HTML 模版
    controller.init(); // 事件处理函数的定义和绑定
  }

  // 数据层
  var model = {
    // 数据管理
    data: {
      a: 0,
      b: 0,
      s: "+",
      r: 0,
    },
    // 对数据进行劫持
    init: function () {
      var _this = this;

      for (const key in _this.data) {
        Object.defineProperty(_this, key, {
          // 当使用 model.a 访问数据就被劫持
          get: function () {
            return _this.data[key];
          },
          set: function (newVal) {
            _this.data[key] = newVal;
            // 去进行更新视图的渲染
            view.render({
              [key]: newVal,
            });
          }
        });
      }
    }
  };

  // 视图层
  var view = {
  };

  // 控制层
  var controller = {
  };

  init();
})();

```

以上代码，我们在`Model`层的`data`中定义了要被劫持的数据，在`init`方法中通过`Object.defineProperty`对数据的`get`和`set`进行相关处理，当`set`某个属性的时候，我们就会去更新视图层。

接下来我们去书写视图层的逻辑：

```js
(function () {
  function init() {
    model.init(); // 组织数据，监听数据（数据代理）
    view.render(); // 组织 HTML 模版 + 渲染 HTML 模版
    controller.init(); // 事件处理函数的定义和绑定
  }

  // 数据层
  var model = {
    // 数据管理
    data: {
      a: 0,
      b: 0,
      s: "+",
      r: 0,
    },
    // 对数据进行劫持
    init: function () {
      var _this = this;

      for (const key in _this.data) {
        Object.defineProperty(_this, key, {
          // 当使用 model.a 访问数据就被劫持
          get: function () {
            return _this.data[key];
          },
          set: function (newVal) {
            _this.data[key] = newVal;
            // 去进行更新视图的渲染
            view.render({
              [key]: newVal,
            });
          }
        });
      }
    }
  };

  // 视图层
  var view = {
    el: "#app",
    template: `
    <div>
        <span class="cla-a">{{ a }}</span>
        <span class="cla-s">{{ s }}</span>
        <span class="cla-b">{{ b }}</span>
        <span>=</span>
        <span class="cla-r">{{ r }}</span>
    </div>
    <div>
        <input type="text" placholder="数字a" class="cal-input a" />
        <input type="text" placholder="数字b" class="cal-input b" />
    </div>
    <div>
        <button class="cla-btn">+</button>
        <button class="cla-btn">-</button>
        <button class="cla-btn">*</button>
        <button class="cla-btn">/</button>
    </div>
    `,
    render: function (mutedData) {
      // 处理数据更改
      // 首次在 init 执行的时候就会执行这里
      if (!mutedData) {
        // 利用正则表达式去匹配模版中的 {{ xxx }}
        // 然后把匹配到的 {{ xxx }} 替换为 Model 层中真实的数据
        this.template = this.template.replace(/\{\{(.*?)\}\}/g, function (node, key) {
          return model.data[key.trim()];
        });

        // 渲染到页面中
        var container = document.createElement("div");
        container.innerHTML = this.template;
        document.querySelector(this.el).appendChild(container);
      } else {
        // 遍历 Model 中的数据替换要更新的节点数据
        for (const key in mutedData) {
          document.querySelector(".cla-" + key).textContent = mutedData[key];
        }
      }
    }
  };

  // 控制层
  var controller = {};

  init();
})();
```

以上代码，我们在`view`层的`template`中给标签都增加了一个类名`cla-*`这是为了我们方便后面去更新对应的数据。

在`render`方法中分别对第一次初始化和`Model`调用进行区分处理，渲染页面。

最后，就是我们的`Controller`层了：

```js
(function () {
  function init() {
    model.init(); // 组织数据，监听数据（数据代理）
    view.render(); // 组织 HTML 模版 + 渲染 HTML 模版
    controller.init(); // 事件处理函数的定义和绑定
  }

  // 数据层
  var model = {
    // 数据管理
    data: {
      a: 0,
      b: 0,
      s: "+",
      r: 0
    },
    // 对数据进行劫持
    init: function () {
      var _this = this;

      for (const key in _this.data) {
        Object.defineProperty(_this, key, {
          // 当使用 model.a 访问数据就被劫持
          get: function () {
            return _this.data[key];
          },
          set: function (newVal) {
            _this.data[key] = newVal;
            // 去进行更新视图的渲染
            view.render({
              [key]: newVal,
            });
          }
        });
      }
    }
  };

  // 视图层
  var view = {
    el: "#app",
    template: `
    <div>
        <span class="cla-a">{{ a }}</span>
        <span class="cla-s">{{ s }}</span>
        <span class="cla-b">{{ b }}</span>
        <span>=</span>
        <span class="cla-r">{{ r }}</span>
    </div>
    <div>
        <input type="text" placholder="数字a" class="cal-input a" />
        <input type="text" placholder="数字b" class="cal-input b" />
    </div>
    <div>
        <button class="cla-btn">+</button>
        <button class="cla-btn">-</button>
        <button class="cla-btn">*</button>
        <button class="cla-btn">/</button>
    </div>
    `,
    render: function (mutedData) {
      // 处理数据更改
      // 首次在 init 执行的时候就会执行这里
      if (!mutedData) {
        // 利用正则表达式去匹配模版中的 {{ xxx }}
        // 然后把匹配到的 {{ xxx }} 替换为 Model 层中真实的数据
        this.template = this.template.replace(/\{\{(.*?)\}\}/g, function (node, key) {
          return model.data[key.trim()];
        });

        // 渲染到页面中
        var container = document.createElement("div");
        container.innerHTML = this.template;
        document.querySelector(this.el).appendChild(container);
      } else {
        // 遍历 Model 中的数据替换要更新的节点数据
        for (const key in mutedData) {
          document.querySelector(".cla-" + key).textContent = mutedData[key];
        }
      }
    }
  };

  // 控制层
  var controller = {
    init: function () {
      var oCalInputs = document.querySelectorAll(".cal-input"),
        	oBtns = document.querySelectorAll(".cla-btn"),
        	inputItem,
        	btnItem;

      // 给所有的输入框框绑定 input 事件
      for (let index = 0; index < oCalInputs.length; index++) {
        inputItem = oCalInputs[index];
        inputItem.addEventListener("input", this.handleInput, false);
      }

      // 给所有的按钮绑定 click 事件
      for (let index = 0; index < oBtns.length; index++) {
        btnItem = oBtns[index];
        btnItem.addEventListener("click", this.handleClick, false);
      }

    },
    // 处理表单输入
    handleInput: function (event) {
      var tar = event.target,
        	value = Number(tar.value),
        	field = tar.className.split(" ")[1]; // 拿到输入框的 a 和 b

      // 然后去操作 model 中对应的值，然后就会触发 set 机制，然后就去渲染 dom
      model[field] = value;

      // ES3 的写法，和 model.r = xxx 一个意思
      // 详见MDN：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with
      with (model) {
        r = eval("a" + s + "b");
      }
    },
    handleClick: function (event) {
      var type = event.target.textContent;
      model.s = type;

      with (model) {
        r = eval("a" + s + "b");
      }
    }
  };

  init();
})();
```

以上代码，我们在`Controller`层分别给输入框和按钮绑定了相关的事件，当事件触发的时候它们就会去操作`Model`的数据，然后就会触发属性的`set`机制，我们在`set`机制里面调用了`View`层的`render`方法，这个时候就产生数据模版，页面也就跟着变化。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1675736751468-c25b6782-9cf7-4500-9b7c-47332acc8b71.gif)

## 🔢 MVC 的缺点

通过上面的案例我们发现，这样的设计模式还不是特别的完美，`View`层本应该只关注于是数据的展示。但里面却包含了`render`方法，我们希望的是有一套驱动，能把数据、视图、事件处理都放在一起集中处理，这就是`ViewModel`。

> MVC 是 MVVM 模型的雏形，MVVM 解决了驱动不内聚的缺点。
>

`Model`层管理数据`data` =>> 通过`ViewModel`层进行连接操作 =>> `View`层关注视图

这样开发者只关注于`Model`和`View`就可以啦，回到`Vue`框架中，`Vue`是关注于视图渲染的。

另外`Vue`允许通过`ref`来直接操作`DOM`，所以严格来说`Vue`并没有完全遵循`MVVM`的模型，`MVVM`是强制`M`和`V`完全分离的！
