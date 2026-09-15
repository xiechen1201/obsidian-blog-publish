---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/00 Webpack 的核心功能/05 devtool 配置/","dg-note-properties":{}}
---

---
---
## 🔢 source map
前端发展到现在，很多时候都不能直接运行源代码，可能需要对源代码进行合并、压缩、转换等操作，真正运行的是转换后的代码。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693967514169-dbd15339-a596-4212-b55b-a65c1446caff.png)

这就给调试带来了很大的困难，因为 bundle.js 文件都是压缩后的。为了解决这个问题，chrome 浏览器率先对 source map 进行了支持，到现在的基本所有浏览器都支持 source map。

source map 实际是一个配置，配置中不仅记录了所有源码内容，还记录了和转换后的代码的对应关系。

浏览器读取 source map 的大致流程：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693967811902-23f1b06a-a120-47ad-b5f5-c0b5bb247060.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693967869443-f177a4d4-c44c-4c4e-abe6-c85f9a8b51ac.png)

总结：

1、source map 作为一种对压缩后代码的调试手段，应该只应用在开发阶段。

2、source map 不应该在生产环境中使用，source map 的文件一般较大，不仅会导致额外的网络传输，还容易暴露原始代码。即便要在生产环境中使用 source map，用于调试真实的代码运行问题，也要做出一些处理规避网络传输和代码暴露的问题。

## 🔢 Webpack 中的 source map
Webpack 也会把源代码进行编译压缩，同样也是无法进行调试的。

所以，Webpack 提供给了 devtool 这个属性让我们配置 source map：

```js
module.exports = {
  mode: "production",
  devtool: 'eval'
};
```

如果 Webpack 的模式为 development 是默认开启了 source map 的，production 模式可以通过 devtool 属性来开启。

devtool 属性有很多的值，但是每个值对构建的速度和构建结果的体积都有区别，

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693968231872-34917006-5c27-437d-aaff-31decde6aa5a.png)

重新构建速度指的是对 Webpack 配置了 watch 监听文件后重新构建的速度。

假如我们的代码是这样滴：

```js
console.log("This is index.js");
require("./a.js");
```

```js
let obj = null;
obj.abc();

console.log("This is a.js");
```

a.js 文件有一个很明显的错误，我看看打包后的结果：

```js
module.exports = {
  mode: "production"
};
```

运行 Webpack 进行打包：

```bash
$ npx webpack
```

然后，就会产生一个 dist 文件夹，我们想要运行 ./dist/mian.js 文件需要手动创建一个 index.html 并且手到导入 ./main.js 文件，最后运行 html 文件：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693968705086-413a5191-6db8-4770-bbb1-0557b7219566.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693968721746-f7813b01-2036-4767-99d3-046d13f0e9b7.png)

可以看到，这已经不是我们之前的代码了，根本无法进行调试。如果我们加上 devtool 属性又是什么样子的呢？

```js
module.exports = {
  mode: "production",
  devtool: "eval"
};
```

最后运行 html 文件：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693968827058-ac7c4252-ae4f-42be-8404-792c4570e12b.png)

这和我们之前的源代码基本是一致的！！！

查看 ./dist/mian.js 文件，发现是通过 sourceURL 进行映射的源代码：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693968901849-e545ab1c-55e1-475d-8571-aadd9fdcde9d.png)

这就是 source map 的魅力！

另外，对于开发环境和生产环境如果使用 source map 都有推荐，详见：

[devtool](https://v4.webpack.docschina.org/configuration/devtool/#%E5%AF%B9%E4%BA%8E%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)
