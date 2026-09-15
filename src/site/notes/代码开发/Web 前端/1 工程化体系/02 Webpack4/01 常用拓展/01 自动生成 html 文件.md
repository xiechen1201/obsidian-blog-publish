---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/01 常用拓展/01 自动生成 html 文件/","dg-note-properties":{}}
---

到目前为止，我们生产的 dist 目录下都没有 HTML 文件，如果想要运行 JS 文件就需要我们每次手动的创建一个 HTML 文件，然后手动导入打包完成的 JS 文件，这样就会非常的麻烦。

我们可以使用`html-webpack-plugin`插件来帮助我们每次打包的时候自动创建一个 HTML 文件且引入打包好的 bundle 文件。

1、安装

```bash
# 太高的版本和 Webpack4 不兼容
$ npm i html-webpack-plugin@4 -D
```

2、进行配置

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
    mode: "development",
    devtool: "source-map",
    output: {
        filename: "[name].[chunkhash:5].js"
    },
    plugins: [
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin()
    ]
};

```

3、运行 Webpack，查看结果

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695017446577-96f8defd-e238-49d0-adde-23a0501b99d9.png)

可以看到，这个插件帮助我们自动创建了一个 index.html 文件且导入了打包后的 JS 文件！

html-webpack-plugin 还允许传递配置参数，详见：

[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)

下面是常见的配置：

1、`template`文件模版

某些情况下，我们可能需要一个 HTML 文件模版，里面写好了一些内容，然后在这个模版的基础上导入打包好的 bundle 文件。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  output: {
    filename: "[name].[chunkhash:5].js"
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template:"./public/index.html"
    })
  ]
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695018186197-72f535e6-8db1-4ebf-8687-d52f1fe368cc.png)

这样就会使用我们的模版来引入 JS 文件！

2、`chunks`使用哪些 chunk 文件

默认情况下，html-webpack-plugin 处理后的 index.html 文件会把所有的 chunk 到进行引入。

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
    mode: "development",
    devtool: "source-map",
    entry: {
        index: "./src/index.js",
        list: "./src/list.js"
    },
    output: {
        filename: "[name].[chunkhash:5].js"
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: "./public/index.html"
        })
    ]
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695018389196-65c8bc7d-b2fc-4838-8f56-ac7be4e73073.png)

这个时候，会把所有的 chunk 到引入进来。

我们可以通过`chunks`来配置只引入某个（些）chunk 文件。

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  entry: {
    index: "./src/index.js",
    list: "./src/list.js"
  },
  output: {
    filename: "[name].[chunkhash:5].js"
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html",
      // 只引入 name 为 index 的chunk
      chunks: ["index"]
    })
  ]
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695018494408-d6074c79-4725-49b5-8ac8-2a75d92ce07f.png)

这样就只会引入我们配置的 chunk 文件啦！

如果我们有多个页面，就可以针对页面来配置引入哪些 chunk：

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  entry: {
    index: "./src/index.js",
    list: "./src/list.js",
    detail: "./src/detail.js"
  },
  output: {
    filename: "[name].[chunkhash:5].js"
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html",
      chunks: ["index"]
    }),
    new HtmlWebpackPlugin({
      template: "./public/list.html",
      chunks: ["list"]
    }),
    new HtmlWebpackPlugin({
      template: "./public/detail.html",
      chunks: ["detail"]
    })
  ]
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695018714509-8202b8ac-6e00-4ddd-bbaa-524a016630bd.png)

但是，为什么最终只有一个 HTML 文件呢？而且还是 detail 页面。

这是因为每一次实例化 html-webpack-plugin 插件的时候都会产生一个新的 HTML 文件，而且每个 HTML 文件都是 index，所以就导致最后生成的 HTML 文件会把之前生成的 HTML 文件替换掉。

3、`filename`文件名称

我们可以通过该属性来对生成的 HTM 文件进行命名。

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
    mode: "development",
    devtool: "source-map",
    entry: {
        index: "./src/index.js",
        list: "./src/list.js",
        detail: "./src/detail.js"
    },
    output: {
        filename: "[name].[chunkhash:5].js"
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: "./public/index.html",
            chunks: ["index"],
            filename: "index.html"
        }),
        new HtmlWebpackPlugin({
            template: "./public/list.html",
            chunks: ["list"],
            filename: "list.html"
        }),
        new HtmlWebpackPlugin({
            template: "./public/detail.html",
            chunks: ["detail"],
            filename: "detail.html"
        })
    ]
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695018925558-d3bdb72d-02d5-44a3-ad6c-360df2f75d3c.png)

这样就会生成 3 个 HTML 文件了，每个文件都引入了对应的 chunk 文件！
