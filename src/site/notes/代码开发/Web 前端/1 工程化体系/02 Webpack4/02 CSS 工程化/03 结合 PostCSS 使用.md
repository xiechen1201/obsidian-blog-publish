---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/02 CSS 工程化/03 结合 PostCSS 使用/","dg-note-properties":{}}
---

---
---
到目前为止可以看得出，CSS 工程化面临着许多的问题，而解决这些问题的方案有多种多样，就是没有一个统一的、且几乎完美的解决方案。

既然有这么多的问题需要处理，为何不出一个工具来集中处理呢？

于是 PostCSS 就基于这个理念出现了。

## 🔢 什么是 PostCSS？
官方网站：

[PostCSS - a tool for transforming CSS with JavaScript](https://postcss.org/)

[https://github.com/postcss/postcss](https://github.com/postcss/postcss)

PostCSS 也是一个编译器，可以把源代码转换为最终的 CSS 代码。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695695332735-5510a98e-d644-4d76-b0d8-889fae05626d.png)

这和 Less、Sass 预处理器不是一样的吗？

Less 和 Sass 同样可以通过自己的 CLI 命令把源码进行转换。

但是 PostCSS 和 Less、Sass 思路又不一样，它其实只负责把代码进行分析之类的事情，然后将分析的结果传递给插件，由插件进行某些处理，整个流程和 Webpack 非常的类似，Webpack 本身仅做依赖分析、抽象语法树分析，其他的操作是靠插件和加载器完成的。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695697362642-e6cd1dab-b0e8-4ba6-8091-6cb02e76734b.png)

## 🔢 使用 PostCSS
要使用 PostCSS 肯定要进行安装：

```bash
$ npm i -D postcss postcss-cli
```

然后通过 CLI 来使用：

```bash
$ npx postcss ./src/style/index.css -o ./dist/index.css
```

这样就可以把源文件进行编译最后进行输出了。

PostCSS 也有自己的配置文件，该配置文件会影响 PostCSS 的某些编译行为，默认的配置名称为 postcss.config.js，因为 PostCSS 运行在 Node 环境下，所以需要使用 CommonJS 规范：

```js
module.exports = {
  map: false, //关闭 source-map
}
```

这样，最终输出的问题就没有 source-map 了。

![配置之前](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695697952160-4fd0615b-cf84-40c5-bb20-daebcbb0a7fe.png)

![配置之后](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695697978377-e2f86913-f508-4d24-ade4-9215d2f507ca.png)

## 🔢 使用插件
PostCSS 的强大之处就是插件的生态笔比较好，所以我们要发挥它的作用，使用一些插件。

PostCSS 的插件市场：

[postcss.parts](https://www.postcss.parts/)

以下是一些常用的的 PostCSS 插件：

## 🔢 postcss-preset-env
在过去没有使用 PostCSS 的时候，往往会使用大量的插件来解决一些问题，这样就导致需要安装非常多的插件、配置插件非常的麻烦。

postcss-preset-env 是一个 PostCSS 的预设环境，大概意思就是它集合了非常多常用的插件，并帮我们完成了基本的配置，我们只需要安装这一个插件即可，就好比安装了一堆插件！

1、安装

```bash
$ npm i -D postcss-preset-env
```

2、配置

```js
module.exports = {
  plugins: {
    "postcss-preset-env": {} // {} 中可以填写插件的配置
  }
};

```

## 🔢 自动添加厂商前缀
某些情况下，我想要使用新的 CSS 功能需要在旧版本浏览器进行兼容，就需要添加一些厂商前嘴来获得支持，例如：

```css
::placeholder {
  color: red;
}
```

该功能在不同的旧版本浏览器中需要书写为:

```css
::-webkit-input-placeholder {
  color: red;
}
::-moz-placeholder {
  color: red;
}
:-ms-input-placeholder {
  color: red;
}
::-ms-input-placeholder {
  color: red;
}
::placeholder {
  color: red;
}
```

如果我们不想书写，想让 PostCSS 自动帮我们进行处理就需要使用 autoprefixer 插件：

```js
const config = {
  plugins: [
    require('autoprefixer')
  ]
}

module.exports = config
```

不过好在 postcss-preset-env 内部已经包含了 autoprefixer，所以我们不再需要单独进行安装，直接使用即可。

如果需要调整兼容的浏览器范围，可以通过下面的方式进行配置：

方法一：直接在 postcss.config.js 中的 postcss-preset-env 进行配置

```js
module.exports = {
  plugins: {
    "postcss-preset-env": {
      browsers: [
        "last 2 version",
        "> 1%"
      ]
    }
  }
}
```

方法二（推荐）：创建一个 .browserslistrc 的文件

```plain
last 2 version
> 1%
```

因为现在工程化很多工具都依赖了 [browserslist](https://github.com/browserslist/browserslist) 工具，所以我们可以把 browserslist 的配置单独提取为一个文件，这样当我们的项目中某些工具需要读取 browserslist 配置的时候都会以这个文件配置文件标准。

方法三：在 package.json 文化的配置中加入 browserslist 书写

```json
{
  "browserslist": [
    "last 2 version",
    "> 1%"
  ]
}
```

一般情况下，大部分网站都使用下面的格式进行书写:

```latex
last 2 version
> 1% in CN
not ie <= 8
```

- last 2 version：表示浏览器的兼容最近期的两个版本
- > 1% in CN：表示匹配中国大于 1% 的人使用的浏览器， in CN 可省略
- not ie <= 8：表示排除掉版本号小于等于 IE8 的浏览器

我们可以通过网站 [https://browserl.ist/](https://gitee.com/link?target=https%3A%2F%2Fbrowserl.ist%2F) 对配置结果覆盖的浏览器进行查询，查询时，多行之间使用英文逗号分割。

## 🔢 未来 CSS 语法
CSS 的某些前沿语法正在制定过程中，没有形成真正的标准，如果希望使用这部分语法，为了浏览器兼容性，需要进行编译， postcss-preset-env 已经包含了这部分的插件。

我们可以通过 postcss-preset-env 的 stage 属性配置，告知 postcss-preset-env 需要对哪个阶段的 CSS 语法进行兼容处理，它的默认值为 2。

一共有 5 个阶段可配置：

- Stage 0: Aspirational - 只是一个早期草案，极其不稳定
- Stage 1: Experimental - 仍然极其不稳定，但是提议已被 W3C 公认
- Stage 2: Allowable - 虽然还是不稳定，但已经可以使用了
- Stage 3: Embraced - 比较稳定，可能将来会发生一些小的变化，它即将成为最终的标准
- Stage 4: Standardized - 所有主流浏览器都应该支持的 W3C 标准

例如：

```js
module.exports = {
  plugins: {
    "postcss-preset-env": {
      stage: 0
    }
  }
};
```

这样尽管某些语法仍处于非常早期的阶段，但是有该插件存在，编译后仍然可以被浏览器识别。

## 🔢 在 Webpack 中使用 PostCSS
1、安装

```bash
$ npm i postcss-loader -D
```

2、进行配置

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.pcss/,
        use: ["style-loader", "css-loader", "postcss-loader"]
      }
    ]
  },
  plugins: [new HtmlWebpackPlugin()]
};
```

这样，当遇到 .pcss 文件的时候，Webpack 会首先把源代码交给 postcss-loader 进行处理，然后再交给 css-loder 和 style-loader。

同样的，当 postcss-loader 执行的时候，它会读取我们工程目录下的 postcss.config.js 作为配置文件，配置内容会影响 PostCSS 的解析行为。
