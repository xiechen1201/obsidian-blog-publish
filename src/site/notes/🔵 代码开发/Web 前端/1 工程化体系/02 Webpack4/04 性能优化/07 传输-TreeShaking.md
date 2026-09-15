---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/1 工程化体系/02 Webpack4/04 性能优化/07 传输-TreeShaking/","dg-note-properties":{}}
---

---
---
什么是 Tree Shaking？

译为“摇晃树”，就好像把树上的果子都摇晃下来一样。

为什么需要 Tree Shaking？

我们在项目开发中会有很多的模块，一个模块导出很多的东西，但是我们可能只会使用一部分，那么我们就可以使用 Tree Shaking 把没用到的代码都移除掉，来达到代码体积的减小。

```js
import { add } from "./myMath"

console.log(add(1,2));
```

```js
export function add(a, b) {
  console.log("add");
  return a + b;
}

export function sub(a, b) {
  console.log("sub");
  return a - b;
}
```

以上代码中，我们只使用了 add 方法，所以希望在打包的时候能把 sub 方法删除掉。

从 Webpack2 开始就开始支持了 Tree Shaking，生产环境自动开启。

<br/>

例如我们对上面的代码进行打包：

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
  mode: "production",
  plugins: [new CleanWebpackPlugin()]
};
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698977558620-61f3b8e1-c627-4110-8e5d-975a18001be0.png)

可以看到，Webpack 只会把 add 方法进行打包却没有 sub 方法。

## 🔢 原理
Webpack 执行构建的时候依然是从入口模块开始解析寻找依赖关系。当解析一个模块的时候，Webpack 会根据 ESModule 导入语句进行判断，该模块依赖了另一个模块的哪个导出。

Webpack 之所以能对 ESModule 进行判断是因为其具有一些的特点：

- 导入导出语句只能是顶层语句
- 模块语句是静态的，在编译的时候就可以确定模块的依赖关系，不像 commonJS 是可以动态加载的
- import 的模块名只能是字符串常量
- import 绑定的变量是不可更改的

所以，这些特点非常有利于分析出稳定的依赖关系。

在具体分析依赖的时候，Webpack 的原则是：保证代码能正常运行，然后尽量的 Tree Shaking。

所以，如果你依赖的是一个导出对象，由于 JS 的动态特性，以及 Webpack 还不够非常的智能，为了保证代码的正常运行，他不会移除对象中的任何信息。

```js
// 导出一个对象
export default {
  add: function (a, b) {
    console.log("add");
    return a + b;
  },
  sub: function (a, b) {
    console.log("sub");
    return a - b;
  }
};
```

1、可以正常的进行 Tree Shaking

```js
import math from "./math";

console.log(math.add(1, 2));

```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698979266943-cb349eb3-4ed5-4922-8990-dd66a451c55e.png)

2、无法正常进行 Tree Shaking

```js
import math from "./math";

console.log(math);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698979309871-914d158b-3661-4329-bab6-623d8b42a50e.png)

因为 Webpack 不知道你后面会不会继续动态的使用 add 和 sub 方法，所以就一块进行了打包。

所以，我们在编写代码的时候要尽量的：

- 使用`export xxx`而不是使用`export default {}`
- 使用`import { xxx } from "xxx"`而不是`import xxx from "xxx"`

<br/>

依赖分析完毕后，Webpack 会根据每个模块每个导出是否被使用，标记其他导出为 dead code（死代码），然后交给代码压缩工具处理，代码压缩工具最终移除掉那些 dead code 代码。

## 🔢 副作用问题
Webpack 在进行 Tree Shaking 的时候始终遵循「一定保证代码能正常运行」的原则。

在满足原则的基础上，再来决定如何进行 Tree Shaking。所以，当 Webpack 无法确定某个模块是否具有副作用的时候，它将默认认为有副作用。

> 什么是副作用函数？
>
> 从名字可以简单的理解就是存在意外的、没有想到的效果，比如人喝药的时候可以治疗疾病但是有可能存在一些意外的效果，例如头晕、恶心等等，这就是副作用。
>
> 回到 JS 中，副作用函数就是可能对函数外部环境造成影响的函数！例如：异步函数、localStorage、对外部数据进行更改
>
> 如果一个函数没有副作用，同时函数的返回结果仅仅依赖函数的参数，那么该函数称为纯函数。
>

所以，导致某些情况并不是预期的那样：

```js
import "./common.js";
```

```js
var n = Math.random();
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698982804097-334ec29e-11a9-45d8-9899-e7a2e4ad2140.png)

以上代码，我们根本没有使用 common.js 里面的代码，但是 Webpack 担心该文件有副作用，所以就正常打包了进来。

如果要解决这个问题，就要标记该文件是没有副作用的，让 Webpack 放心大胆的去打包：

方式一：在 package.json 文件中进行配置`sideEffects`为布尔值

```json
{
  "name": "_demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server",
    "build": "webpack",
    "build:dll": "webpack --config ./webpack.dll.config.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    // ...
  },
  "dependencies": {
    // ...
  },
  "sideEffects": false
}
```

`false`表示所有的模块都没有副作用，但是这种方式会影响某些 CSS 文件的导入。

方式二：在 package.json 文件中进行配置`sideEffects`为数组

```json
{
  "name": "_demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server",
    "build": "webpack",
    "build:dll": "webpack --config ./webpack.dll.config.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    // ...
  },
  "dependencies": {
    // ...
  },
  "sideEffects": ["!src/common.js"]
}
```

表示只有 ./src/common.js 文件没有副作用，其他文件都有副作用。

> 这种方式我们一般不处理，通常是一些第三方库在它们自己的 package.json 中标注。
>

## 🔢 CSS Tree Shaking
一般来说，CSS 是无法完成 Tree Shaking 的，因为 CSS 和 ESModule 没有任何的关系，但是我们可以借助 purgecss-webpack-plugin 插件来帮助我们实现类似的效果。

例如我们只使用`.box`这个选择器：

```css
.wrapper {
  border: 1px solid blueviolet;
  border-radius: 50px;
}

.box {
  width: 100px;
  height: 100px;
  background-color: aquamarine;
}
```

```css
import "./index.css"

let oDiv = document.createElement("div");
oDiv.className = "box";
oDiv.innerHTML = "Hello World";
document.body.appendChild(oDiv);
```

然后安装并配置 Webpack 的配置文件：

```bash
$ npm i purgecss-webpack-plugin -D
```

```js
const path = require("path");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const { PurgeCSSPlugin } = require("purgecss-webpack-plugin");

module.exports = {
  mode: "production",
  module: {
    rules: [
      {
        test: /\.css/,
        use: [MiniCssExtractPlugin.loader, "css-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new MiniCssExtractPlugin(),
    new PurgeCSSPlugin({
      // 和 html 文件中的 CSS 选择器进行对比，没有用到的选择器就删除
      paths: [
        path.resolve(__dirname, "./public/index.html"),
        path.resolve(__dirname, "./src/index.js")
      ]
    })
  ]
};
```

最后执行构建，可以看到 main.css 文件只剩下 .box 这个选择器了！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698993604680-1b99fd5d-63ea-4cf0-9bd7-26a3b9c953ae.png)

但是这样每一个文件都去匹配会显的非常的麻烦，我们可以使用`glob`语法来实现匹配：

```js
const path = require("path");
const glob = require("glob");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const PurgeCSSPlugin = require("purgecss-webpack-plugin");

const PATH = {
  src: path.resolve(__dirname, "src")
};

module.exports = {
  mode: "production",
  module: {
    rules: [
      {
        test: /\.css/,
        use: [MiniCssExtractPlugin.loader, "css-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new MiniCssExtractPlugin({
      filename: "[name].css"
    }),
    new PurgeCSSPlugin({
      // 和 html 文件中的 CSS 选择器进行对比，没有用到的选择器就删除
      /* paths: [
        path.resolve(__dirname, "./public/index.html"),
        path.resolve(__dirname, "./src/index.js")
      ] */
      // 匹配 src 下所有的文件
      paths: glob.sync(`${PATH.src}//*`)
    })
  ]
};
```

<br/>warning
⚠️ 注意

但是，该库对 CSS Module 是无效的，因为 CSS Module 返回的是转换后的类名，它无法匹配到正确的内容！

<br/>
