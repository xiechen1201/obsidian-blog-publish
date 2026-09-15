---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/1 工程化体系/02 Webpack4/02 CSS 工程化/01 Webpack 打包 CSS 文件/","dg-note-properties":{}}
---

---
---
要在 Webpack 中打包 CSS 代码，就需要也把 CSS 文件视作一个模块，然后通过相关的 Loader 或者 Plugin 来进程处理，因为 Webpack 本身是可以读取文件内容的，但是无法把 CSS 代码转换为 AST 语法树。

## 🔢 css-loader
css-loader 的作用就是把 CSS 代码转换为 JS 的代码，处理原理很简单，就是把 CSS 代码作为字符串返回。

例如：

```css
.red{
    color:"#f40";
    background:url("./bg.png")
}
```

通过 css-loader 后处理：

```js
var import1 = require("./bg.png");
module.exports = `.red{
    color:"#f40";
    background:url("${import1}")
}`;
```

> 以上代码是简化版本，并不是真的就这么简单，css-loader 的处理流程是很复杂的，同时也会导出更多的东西！
>

这样一来，经过 Webpack 的后续处理，会把依赖`./bg.png`添加到模块列表，然后再将代码转换为

```js
var import1 = __webpack_require__("./src/bg.png");
module.exports = `.red{
    color:"#f40";
    background:url("${import1}")
}`;
```

再比如：

```js
@import "./reset.css";
.red{
    color:"#f40";
    background:url("./bg.png")
}
```

转换后的结果：

```js
var import1 = require("./reset.css");
var import2 = require("./bg.png");
module.exports = `${import1}
.red{
    color:"#f40";
    background:url("${import2}")
}`;
```

总结，css-loader 干了什么：

1、将 CSS 文件的内容作为字符串导出

2、将 CSS 中的其他依赖作为 require 导入，以便 Webpack 分析依赖

那么如何使用 css-loader 呢？

1、安装

```bash
# 高版本和 Webpack4 不兼容
$ npm i css-loader@5 -D
```

2、进行配置

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["css-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    })
  ]
};
```

3、编写源代码

```js
import style from "./style/index.css";

console.log(style)
```

```js
#app {
  width: 100px;
  height: 100px;
  border-radius: 10px;
  background-color: #444;
}
```

4、最后我们运行编译

```bash
$ npx webpack
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695631840912-b0a9472d-5450-4f45-a97d-46314a0e12ce.png)

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695631861349-28a68173-b0f5-4f7e-bc2f-5c50fd36dba0.png)

然后，我们却发现 dist 目录下根本没有 CSS 文件，且页面中也并没有相应的样式。

但是，我们却能在控制台看到 css-loader 处理后返回的内容，是一个数组：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695631939002-498fc665-5dd1-43cf-8840-e37c2b09fb94.png)

再看 dist/main.js 文件发现，index.css 文件是作为一个模块被打包到了 main.js 文件内部：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695631999576-05cf2cdd-d59d-4531-8da6-028771d587fe.png)

所以，我们知道了，css-loader 真的只是把 CSS 代码导出。

如果，CSS 文件内🈶又导入了一张图片会返回什么呢？

```css
#app {
  width: 100px;
  height: 100px;
  border-radius: 10px;
  /* background-color: #444; */
  background-image: url(../imgs/template.jpeg);
}
```

同样的，css-loader 会把图片当作是一个模块，所以我们还需要配置对于的 file-loader 防止 Webpack 解析错误：

```css
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["css-loader"]
      },
      {
        test: /\.jpeg$/,
        use: ["file-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    })
  ]
};
```

最后打包运行 HTML 文件，发现 css-loader 的返回内容加载的是解析后的图片地址：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695632296016-1c17dc5c-6e62-4973-b74b-9d0871c83a29.png)

那么，css-loader 只返回了一个数组，如何应用到页面上呢？这就需要用到第二个 Loader 了：style-loader!

## 🔢 style-loader
由于 css-loader 仅提供了将 CSS 转换为字符串导出的能力，剩余的事情要交给其他 Loader 或 Plugin 来处理。

style-loader 可以将 css-loader 转换后的代码进一步处理，将 css-loader 导出的字符串加入到页面的style 元素中。

1、安装

```bash
# 高版本和 Webpack4 不兼容
$ npm i style-loader@2 -D
```

2、配置

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      },
      {
        test: /\.jpeg$/,
        use: ["file-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    })
  ]
};

```

只需要在原先 css-loader 前面追加 style-loader 即可，因为 Loader 的执行顺序是从后往前执行的，我们期望的是先把 CSS 代码交给 css-loader 进行处理，然后再交给 style-loader 进行处理。

3、运行打包，查看结果

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695632675351-820d3d7b-ff2e-4bde-ab4a-2e14cb75e1b1.png)

可以看到，style-loader 已经帮我们把代码添加到 style 标签内部！

另外，经过 style-loader 处理后的代码，将不再导出任何内容：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695632761462-f9851aaf-1ba6-42c7-bb90-e955ac3eb0f2.png)

## 🔢 开启 css module
上一章我们简单提了一下 css module 功能，下面就详细看看如何使用。

css-loader 默认就支持 css-module，只要给它传递一个`modules: true`的属性即可开启。

当开启 css-module 后，css-loader 会把我们原先的类名进行重新命名为一个 hash 值来打包类名唯一的目的：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695633064450-f7aafebb-bb75-4d0f-9f8b-86f36e56792c.png)

由于 hash 值是根据模块路径和类名生成的，因此，不同的 CSS 模块，哪怕具有相同的类名，转换后的hash 值也不一样。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695633101392-29c9550a-20d8-4676-b25b-8f60a93cb65a.png)

那么，问题来了，css-loader 把类名都重新命名了，开发者只知道自己书写的类名，并不知道最终会生成的类名是啥，该如何应用到元素上呢？

为了解决这个问题，css-loader 会导出原类名和最终类名的对应关系，该关系是通过一个对象描述的：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695633319453-eb37bfca-aa2d-4d61-9a54-cc94d7a59e31.png)

下面就一起看看到底如何开启：

1、更改配置文件

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.css$/,
        // use: ["style-loader", "css-loader"]
        use: ["css-loader?modules"] // 传递参数开启 css-module
      },
      {
        test: /\.jpeg$/,
        use: ["file-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    })
  ]
};

```

```js
import style from "./style/index.css";

console.log(style)
console.log(style.toString())
```

2、运行打包，查看效果

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695633485680-3e004276-991d-40ca-a9ef-79422e93b8b0.png)

可以看到，css-loader 通过 locals 属性把对应关系返回了。

如果，我们同时开启了 style-loader，那么 style-loader 只会把映射结果返回，其他信息屏蔽。

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.css$/,
        // use: ["style-loader", "css-loader"]
        use: ["style-loader", "css-loader?modules"]
      },
      {
        test: /\.jpeg$/,
        use: ["file-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    })
  ]
};

```

```js
import style from "./style/index.css";

for (const key in style) {
  // 获取元素并替换 ID（类名也是同样的操作）
  document.querySelector("#" + key).id = style[key];
}
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695633934612-9226dc34-5279-4ac8-b527-f69bfa5effd7.png)

依然是正常执行！

## 🔢 其他操作
某些类名是全局的、静态的，不需要进行转换，仅需要在类名位置使用一个特殊的语法即可：

```css
:global(.main){
  ...
}
```

使用了 global 的类名不会进行转换，相反的，没有使用 global 的类名，表示默认使用了 local：

```css
:local(.main){
    ...
}
```

使用了 local的 类名表示局部类名，是可能会造成冲突的类名，会被 css module 进行转换。

<br/>warning
⚠️ 注意

- css module 往往配合构建工具使用
- css module 仅处理顶级类名，尽量不要书写嵌套的类名，也没有这个必要
- css module 仅处理类名，不处理其他选择器
- css module 还会处理 id 选择器，不过任何时候都没有使用 id 选择器的理由
- 使用了 css module 后，只要能做到让类名望文知意即可，不需要遵守其他任何的命名规范

<br/>

## 🔢 使用预处理器 Less
因为预处理 Less 的文件是一个 .less 的后缀名，Webpack 解析的时候又不知道如何解析为 AST 了，需要 Less 为我们提供了对应的 less-loader 专门用来出来 less 文件。

1、安装

```bash
# 高版本和 Webpack4 不兼容
$ npm i less-loader@7 -D
```

2、编写一段 Less 代码

```css
@successColor: #67c23a;
@warningColor: #e6a23c;
@errorColor: #f56c6c;
@infoColor: #909399;

.bordered {
  border-top: dotted 1px black;
  border-bottom: solid 2px black;
}

.a {
  color: @successColor;
}

.b {
  background-color: @infoColor;
  .bordered();

  .c {
    color: @warningColor;
  }
}
```

```css
import "./style/demo.less";
```

3、配置 Loader

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.css$/,
        // use: ["style-loader", "css-loader"]
        use: ["style-loader", "css-loader?modules"]
      },
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader", "less-loader"]
      },
      {
        test: /\.jpeg$/,
        use: ["file-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    })
  ]
};
```

4、打包查看效果

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695635230132-4f962780-ef73-487a-b1ef-875953676983.png)

可以看到 Less 代码被正常转换为 CSS 代码，因为因为 Less 文件首先会经过 less-loder 的处理，然后交给 css-loader 进行处理，最后由 style-loader 把代码添加到 style 标签内部。
