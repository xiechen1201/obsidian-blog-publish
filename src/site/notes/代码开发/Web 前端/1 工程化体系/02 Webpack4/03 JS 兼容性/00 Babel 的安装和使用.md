---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/03 JS 兼容性/00 Babel 的安装和使用/","dg-note-properties":{}}
---

## 简介
官方文档：

[Babel · Babel](https://babeljs.io/)

民间中文：

[Babel 中文文档 | Babel中文网 · Babel 中文文档 | Babel中文网](https://www.babeljs.cn/)

到目前为止，仍然存在的问题就是不同版本的浏览器对 ES 的兼容程度不一样，这就导致开发者如果要兼容低版本的浏览器就要编写兼容的代码。

Babel 是一个编译器，它可以把同一份代码转换为兼容各种浏览器的代码，减少了开发者的痛苦。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695782535136-63909906-6c4f-4f8a-ad27-c26e45b4414d.png)

Babel 和 PostCSS 一样，本身只会进行语法分析，如何转换都需要交给插件和预设进行处理。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695782594097-327b6ce8-d887-471f-ac99-fb8947baf929.png)

## 安装 & 使用
Babel 可以和构建工具结合使用，也可以通过其 CLI 进行独立使用。

本文只看独立使用的情况，如果要独立使用还需要安装 Babel 的 CLI 工具：

```bash
$ npm i -D @babel/core @babel/cli
```

- @babel/core 是 Babel 的核心库，提供了编译所需要的所有 API
- @babel/cli 是 Babel 的 CLI 工具，他可以调用 core 的 API 来完成编译

然后我们就可以通过 CLI 来使用 Babel：

```bash
# 按文件编译
$ babel ./src/index.js -o ./dist/result.js

# 按目录编译
$ babel src -d dist
```

运行结果：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695783145229-c169bc61-ad0f-49e2-ab9d-9d847d4b403b.png)

可以看到 Babel 转换后的代码根本没有变化，这是因为 Babel 执行的时候还要依赖 Babel 的插件和预设来协助。

## 配置文件
我们通过新建一个 babel.config.js 文件：

```js
module.exports = {
  presets: [], // 配置预设
  plugins: [] // 配置插件
};
```

## 插件
插件就是帮助 Babel 来完成一些事情，例如我们想要把源代码中的箭头函数进行转换：

安装：

```bash
$ npm i @babel/plugin-transform-arrow-functions -D
```

配置：

```js
module.exports = {
  presets: [],
  plugins: ["@babel/plugin-transform-arrow-functions"]
};
```

结果：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695784131931-e036a0d4-cb1a-424c-b380-4445b3191a7c.png)

当然，我们还可以配置更多的插件，然后转换不同的语法：

```js
module.exports = {
  presets: [],
  plugins: [
    "@babel/plugin-transform-arrow-functions",
    "@babel/plugin-transform-classes",
    "@babel/plugin-transform-for-of"
    // ...
  ]
};
```

插件的执行顺序是从前往后执行的！

更多插件详见：

[Plugins List · Babel 中文文档 | Babel中文网](https://www.babeljs.cn/docs/plugins-list)

## 预设
预设是多个插件的集合，内部集成了很多常用的插件，不再需要我们一个一个进行手动的安装。

安装：

```bash
$ npm i @babel/preset-env -D
```

配置：

```js
module.exports = {
  presets: [
    "@babel/preset-env"
  ],
  plugins: [
    // "@babel/plugin-transform-arrow-functions",
    // "@babel/plugin-transform-classes",
    // "@babel/plugin-transform-for-of"
    // ...
  ]
};
```

同样的，预设也可以是多个：

```js
module.exports = {
  presets: [
    "@babel/preset-env",
    "@babel/preset-react",
  ]
};
```

和插件不一样的是，预设是从后往前执行的。

如果我们希望预设要根据不同的浏览器版本范围进行编译，就可以给预设传递参数：

```js
module.exports = {
  presets: [
    // 改成为二维数组的形式
    [
      "@babel/preset-env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1",
          "ie": "8"
        }
      }
    ]
  ],
  plugins: [
    // "@babel/plugin-transform-arrow-functions",
    // "@babel/plugin-transform-classes",
    // "@babel/plugin-transform-for-of"
    // ...
  ]
};
```

当然，我们可以和 PostCSS 一样，创建一个 .browserslistrc 文件，然后在文件内编写浏览器的范围。

浏览器的范围会影响编译的结果，如果你要兼容的浏览器的版本比较高，那么源码很有可能不会被转换！

运行：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695785345709-cfc6f533-d194-4578-b1b7-db41ad5da8c2.png)

可是，我们发现`Promise`这样的 API 却没有被转换，这是因为预设只会「转换语法」，而 API 是新的函数，无法进行转换，这就需要使用 Babel 的「垫片 Polyfill」概念。

## Polyfill
什么是 Polyfill？

它和插件不一样，插件能帮我们进行语法的转换，把新的语法转换为旧的语法，让各个浏览器进行兼容。

但是新的 API 不一样，这些 API 在低版本浏览器中没有被实现，所以就需要 Babel 去实现这些 API 函数，例如`Promise()`、`Map()`、`Set()`。

所以，Polyfill 就是垫片，帮我们把路上的坑“垫平”。

在 Babel 7.4.0 之前，要开启 polyfill 需要安装使用 @babel/polyfill，但是 7.4.0 后面的版本，官方已经不建议使用了，而是使用 core-js 和 regenerator runtime 来完整的模拟 ES6+ 的环境。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695785616128-9e41f86a-e4ef-4372-a9c1-9c5330d082c9.png)

幸运的是，如果想要开启 Polyfill 我们直接在配置文件中加上`useBuiltIns: "usage"`和`"corejs": 3`配置即可，因为编写 Polyfill 的时候使用的 core@2 的版本，我们需要手动告诉 Babel 要使用的版本：

```js
module.exports = {
  presets: [
    // 改成为二维数组的形式
    [
      "@babel/preset-env",
      "useBuiltIns": "usage",
      "corejs": 3,
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1",
          "ie": "8"
        }
      }
    ]
  ],
  plugins: [
    // "@babel/plugin-transform-arrow-functions",
    // "@babel/plugin-transform-classes",
    // "@babel/plugin-transform-for-of"
    // ...
  ]
};
```

`usebuiltins`的值默认为`false`，表示不注入任何新的 API，将其设置为`usage`，表示根据 API 的使用情况，按需导入 API。

另外因为 Polyfill 依赖的是 core-js，所以我们还需要安装 core-js:

```bash
$ npm i -D core-js
```

这个时候再运行 Babel 就可以看到导入了 core-js 的内容用来填充缺失的 API：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695786377318-5d661716-9c81-40e5-a187-855679dffab0.png)
