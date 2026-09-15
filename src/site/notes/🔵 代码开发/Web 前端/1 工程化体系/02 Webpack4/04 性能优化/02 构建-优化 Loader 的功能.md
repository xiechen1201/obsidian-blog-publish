---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/1 工程化体系/02 Webpack4/04 性能优化/02 构建-优化 Loader 的功能/","dg-note-properties":{}}
---

---
---
## 🔢 进一步优化 Loader 的应用范围
如何进行优化？

还是用 jquery 来举例，我们的项目中那往往会使用 babel-loader 对 JS 的代码进行处理，将 ES6+ 的代码转换为低版本浏览器可以运行的代码。然后 jquery 本身就是使用 ES5 编写的，或者它在发布的时候通常已经进行了 JS 的兼容处理，所以我们的项目中就没有必要再对这些代码进行转换了！

如何进行优化呢？

使用`module.rules[].exclude`进行排除某些模块，使用`module.rules[].include`进行包含某些模块：

```js
module.exports = {
  mode: "development",
  module: {
    rules: [
      {
        test: /\.js/,
        // 配置对所有 node_modules 下的模块都不进行 loader 的转换
        exclude: /node_modules/,
        use: ["babel-loader"]
      }
    ]
  }
};
```

然后运行结果，可以看到配置前后打包速度的对比：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698743842193-fc82295c-5cd2-442a-85bf-c9c2bb59c613.png)

`module.rules[].exclude`和`module.noParse`属性配置不会产生冲突，因为`module.noParse`影响的是不对模块进行解析，而`module.rules[].exclude`配置的是不对模块代码进行转换，它们负责的是两个阶段的工作。

<br/>

## 🔢 缓存 Loader 的结果
某些情况下，文件的内容没有进行变化，但是仍然会经过 Webpack Loader 的处理，结果就是白白进行解析，解析的结果没有一点变化。

于是，我们就可以想办法让经过一次 Loader 的转换，然后把转换的结果缓存起来，当下次编译的时候直接读取缓存而不是重新转换。

cache-loader 就可以实现这个功能：

```bash
$ npm i -D cache-loader
```

```js
module.exports = {
  mode: "development",
  module: {
    rules: [
      {
        test: /\.js/,
        // exclude: /node_modules/,
        // 先经过 babel-loader 对 JS 代码紧张转换，再经过 cache-loader 把结果进行缓存
        use: ["cache-loader", "babel-loader"]
      }
    ]
  }
};
```

然后我们看一下配置前后的耗时对比：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698744798902-61e631e2-c194-4c3e-b676-680e1df7d61d.png)

可以看到，构建的时间确实缩减了。

但是，大家可能会有一个疑问：Loader 的执行顺序不是从后往前执行的吗？虽然第一次执行顺序没有问题，但是当第二次的时候不是依然会从 babel-loader 开始执行吗？

其实不然，Loader 在运行的过程中还包含一个过程 pitch（扔）。我们都知道 Loader 实际上就是一个函数，函数接受源文件内容作为参数。

其实 Loader 函数还可以添加一个 pitch 属性，在 Webpack 构建的时候参与

```js
function LoaderTest(sourceCode) {}

// 设置一个 pitch 属性，是一个函数
LoaderTest.pitch = function (filepath) {
  // 可返还内容或者不返回
  // 如果返回，源代码
};

module.exports = LoaderTest;
```

使用下面的案例进行讲解：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698745131984-431c1506-6c20-41a3-b147-29bf3bfc2752.png)

当 Webpack 构建运行 Loader 的时候，其实前面还有一个 pitch 的过程。

例如，运行 Loader 的时候 Webpack 会把 ./src/index.js 文件路径交给 Loader1.pitch 函数运行，如果 Loader1.pitch 执行完成且没有返回结果，那么继续把路径交给 Loader2.pitch 进行处理，以此类推，直到最后读取文件的资源，开始 Loader3、Loader2、Loader1 这样的流程进行转换处理。

pitch 函数可以通过是否返回内容来控制下一步执行到哪里，例如 Loader1.pitch 返回了内容，那么就直接结束了，如果 Loader2.pitch 返回了内容，那么就会交给 Loader1 进行处理。

所以，cāche-loader 放在数组的第一项，是为了让 cache-loader.pitch 直接返回，这样就不会执行 babel-loader 了，而 cache-loader.pitch 函数内部就是看是否具有缓存来决定是否返回内容！

## 🔢 为 Loader 开启多线程
另外我们可以借助 thread-loader 会开启一个多线程，线程池中包含适量的线程。

例如有 10 个 JS 文件需要解析，过程会比较慢，因为要依次进行解析，利用多线程进行并行处理。

```bash
$ npm i thread-loader -D
```

```js
module.exports = {
  module: {
    rule: [
      {
        test: /.js$/,
        // "thread-loader" 后续的 Loader 会放到线程池中运行
        use: ["cache-loader", "thread-loader", "babel-loader"]
      }
    ]
  }
};
```

由于后续的 loader 会放到新的线程中，所以，后续的 loader 不能：

- 使用 Webpack Api 生成文件
- 无法使用自定义的 Plugin Api
- 无法访问 Webpack Options

所以，所以后面只能是一些纯粹的转换代码的功能，不依赖 Webpack 功能的 Loader。

<br/>warning
⚠️ 注意

开启和管理线程需要消耗时间，在小型项目中使用 thread-loader 反而会增加构建时间。

<br/>
