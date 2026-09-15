---
{"dg-publish":true,"permalink":"//web/1/02-webpack4/04/11-bundle/","dg-note-properties":{}}
---

Bundle 分析简单说就是对打包的结果进行分析，它本身不会进行任何优化操作，只是方便开发者进行结果的分析，然后间接的进行优化。

需要安装插件：

```bash
$ npm i webpack-bundle-analyzer -D
```

进行配置使用：

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const WebpackBundleAnalyzer = require('webpack-bundle-analyzer');

module.exports = {
  mode: 'production',
  plugins: [
    new CleanWebpackPlugin(),
    new WebpackBundleAnalyzer.BundleAnalyzerPlugin({
      analyzerMode: 'static',
    }),
  ],
};
```

`analyzerMode`属性可以是下面的值：

- server，默认值，启动一个服务器查看编译的结果
- static，生成一个 report.html 文件，手动在浏览器中查看编译的结果

然后正常执行`npx webpack`就会在生成一个 HTML 页面，运行页面查看结果：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698999338521-16d7ffbb-d17f-4c61-aaf1-a4320e471dbb.png)

页面就会显示出每个包的大小，开发者可以根据实际情况进行分包、Tree Shaking 等优化手段。
