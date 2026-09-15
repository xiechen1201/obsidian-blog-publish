---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/04 性能优化/09 传输-gzip/","dg-note-properties":{}}
---

---
---
一般来说，gzip 和 Webpack 没有直接的关系，那是客户端和服务端的事情。

压缩文件的格式有很多种，gzip 是其中的一种。

## 🔢 B/S 结构中的压缩传输
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1699000488522-a20504a9-8685-4000-b6fa-27edb4770aab.png)

整个过程大致如下：

- 浏览器告诉服务器支持哪些压缩格式解压
- 服务收到请求头后，经过代码对象源文件进行压缩然后返回到浏览器
- 浏览器根据响应头的内容，使用响应的格式进行解码，然后呈现出内容

优点：传输效率得到极大的提升，但并不是绝对的。

缺点：服务器压缩需要时间，客户端的解压需要时间。

## 🔢 使用 Webpack 进行压缩
Webpack 的压缩只是在打包完成后对文件进行压缩，本质上就是替换了服务端压缩的时间。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1699000795926-4da94c2a-07e9-4977-a0ce-96bbf3cf0f7a.png)

要使用 Webpack 进行压缩，需要安装 compression-webpack-plugin 插件：

```bash
# 高版本和 Webpack4 不兼容
$ npm i compression-webpack-plugin@6 -D
```

然后进行配置：

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const CompressionWebpackPlugin = require('compression-webpack-plugin');

module.exports = {
  mode: 'production',
  plugins: [
    new CleanWebpackPlugin(),
    new CompressionWebpackPlugin()
  ]
};
```

然后执行构建：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1699000982886-8ab52ab1-2ecb-4997-a7bd-7faca0bbabc2.png)

可以看到生成了对应的压缩包。

<br/>warning
⚠️ 注意

这样的方式虽然节省了服务端对源文件进行压缩的过程，但是也丢失了灵活性，服务端无法针对特殊的情况进行不同格式的压缩。

<br/>
