---
{"dg-publish":true,"permalink":"//web/1/02-webpack4/03-js/01-webpack-babel/","dg-note-properties":{}}
---

在 Webpack 中使用 Babel 可以说是非常的简单，Babel 也提供了一个 babel-loader 让 Webpack 转换 JS 的代码，然后运行 babel-loader 的时候会读取 Babel 的配置文件，根据配置文件来决定如何转换 JS 代码。

1、安装

```bash
$ npm i @babel/core @babel-loader -D
```

2、配置 Webpack

```js
module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ["babel-loader"]
      }
    ]
  }
};
```

3、配置 Babel

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ]
}
```

4、执行 Webpack，查看结果

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/03%20JS%20%E5%85%BC%E5%AE%B9%E6%80%A7/_assets/1695787030622-14cc3af2-9452-4ff0-835b-404b385463f3.png)

可以看到，dist/main.js 文件里面打包进了好多代码，这些代码都是 core-js 里面的内容，都是为了补充`Map()`函数！
