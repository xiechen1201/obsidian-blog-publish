---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/01 常用拓展/06 Webpack 内置插件/","dg-note-properties":{}}
---

内置插件就是不需要安装就可以直接使用的插件，内置插件都是作为 Webpack 的静态方法存在的。

```js
const webpack = require("webpack");
new webpack.PluginName(options);
```

## DefinePlugin
该插件用来定义全局的常量，使用插件通常定义一些常量值：

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
    mode: "development",
    devtool: "source-map",
    plugins: [
        new CleanWebpackPlugin(),
        new webpack.DefinePlugin({
            PI: `Math.PI`, // PI = Math.PI
            VERSION: `"1.0.0"`, // VERSION = "1.0.0"
            DOMAIN: JSON.stringify("duyi.com")
        })
    ]
};
```

```js
console.log(PI)
console.log(VERSION)
console.log(DOMAIN)
```

这样一来，在源码中，我们可以直接使用插件中提供的常量，当 Webpack 编译完成后，会自动替换为常量的值。````内填写的不是字符串，而是具体代码。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695094178923-aa9edff3-ceaa-4351-accc-721df881c1fc.png)

## BannerPlugin
它可以为每个 chunk 生成的文件头部添加一行注释，一般用于添加作者、公司、版权等信息。

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  plugins: [
    new CleanWebpackPlugin(),
    new webpack.DefinePlugin({
      PI: `Math.PI`, // PI = Math.PI
      VERSION: `"1.0.0"`, // VERSION = "1.0.0"
      DOMAIN: JSON.stringify("duyi.com")
    }),
    new webpack.BannerPlugin({
      banner: `
                hash:[hash]
                chunkhash:[chunkhash]
                name:[name]
                author:xiechen
                corporation:testStr
            `
    })
  ]
};

```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695094328627-52a2e2f4-9ebb-4878-ad9a-0c3c33ef600f.png)

## ProvidePlugin
自动加载模块，而不必到处 import 或 require

```js
const webpack = require("webpack");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
    mode: "development",
    devtool: "source-map",
    plugins: [
        new CleanWebpackPlugin(),
        new webpack.DefinePlugin({
            PI: `Math.PI`, // PI = Math.PI
            VERSION: `"1.0.0"`, // VERSION = "1.0.0"
            DOMAIN: JSON.stringify("duyi.com")
        }),
        new webpack.BannerPlugin({
            banner: `
                hash:[hash]
                chunkhash:[chunkhash]
                name:[name]
                author:xiechen
                corporation:testStr
            `
        }),
        new webpack.ProvidePlugin({
            $: "jquery"
        })
    ]
};

```

```js
console.log($);
```

只要源代码中使用了`$`这个变量，Webpack 就会自动帮我们导入 jquery。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/01%20%E5%B8%B8%E7%94%A8%E6%8B%93%E5%B1%95/_assets/1695094535235-bcc4d0f0-5588-4f76-ba39-4ab937d9c8df.png)
