---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/02 CSS 工程化/02 抽离 CSS 文件/","dg-note-properties":{}}
---

---
---
当目前为止，我的 CSS 文件作为模块依赖都是被打包到 JS 文件内部的，最后再由 style-loader 动态的添加到页面上去。

然而，某些情况下我们希望能够把 CSS 代码单独提取出来一个 .css 文件，这就需要使用 mini-css-extract-plugin 插件了。

mini-css-extract-plugin 提供了 1 个 Plugin 和 1 个 Loader。

- 提供的 Loader 负责记录要生成的 CSS 文件内容，同时导出开启 css-module 时的样式对象
- 提供的 Plugin 负责生成 CSS 文件

1、安装

```bash
# 高版本和 Webpack4 不兼容
$ npm i -D mini-css-extract-plugin@1
```

2、进行配置

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.less$/,
        // use: ["style-loader", "css-loader?modules", "less-loader"]
        use: [MiniCssExtractPlugin.loader, "css-loader?modules", "less-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    }),
    new MiniCssExtractPlugin()
  ]
};

```

3、运行 Webpack，查看效果

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695710099328-d4c67a37-254c-4ee3-ab8f-0e64b3663c62.png)

![开启 css-module 时候的对应关系](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695710114096-667527b5-cae5-4f07-8132-b2d0a0c16852.png)

默认情况下，CSS 文件的名称是根据入口 chunk 来决定的，例如图片中的 main.css 是因为 Webpack 默认的 chunk 就是`mian: "./src/index.js"`，我们也可以给 mini-css-extract-plugin 传递一个 filename 属性来更改名字：

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.less$/,
        // use: ["style-loader", "css-loader?modules", "less-loader"]
        use: [MiniCssExtractPlugin.loader, "css-loader?modules", "less-loader"]
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
    }),
    new MiniCssExtractPlugin({
      filename: "css/[name].[contenthash:5].css"
    })
  ]
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695710351358-1babf8ba-467d-470a-ac70-3221552027d2.png)
