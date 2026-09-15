---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/1 工程化体系/02 Webpack4/00 Webpack 的核心功能/08 Loader/","dg-note-properties":{}}
---

---
---
Webpack 做的事情仅仅是分析各个模块的依赖关系，然后产生一个资源列表，最终打包生成到指定的文件中去。而更多的功能是需要借助 Loader 和 Plugin 来完成的！

## 🔢 执行流程
Loader 本质上就是一个普通的函数，它的作用就是把源码字符串转换为另一个源码字符串后返回！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694401020510-35c42081-8e9c-4f75-b029-6de3917ed955.png)

Loader 会在 Webpack 的模块解析过程中被调用，然后得到最终的源码。

回顾 Webpack 解析 chunk 的流程：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694401096446-98311108-ffcd-45dd-a00f-a0670e8470be.png)

而 Loader 就是在形成 AST 语法树之前执行的：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694401135928-75f4e9c1-aa62-415a-b094-ef48a0bdd926.png)

处理 Loader 的流程：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694401204876-0c416b7d-9304-45b9-8749-05d73ebce4d9.png)

## 🔢 从一行代码开始
例如我们的 ./src/index.js 文件有这么一段代码：

```js
变量 a = 1;
```

这个时候 Webpack 肯定是无法把这段代码解析为 AST 语法树的，因为它不认识“变量”是什么！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694401407854-a52be617-d31c-4675-abfa-61b0c51f6d05.png)

我们可以让 Webpack 把这段代码交给 Loader，让 Loader 处理完后在再返回给 Webpack。

Loader 的配置遵守如下的规则：

```js
module.exports = {
  mode: "development",
  // 模块
  module: {
    // 模块的匹配规则
    rules: [
      // 规则1
      // 规则2
      // 规则3
      {
        // test 是一个正则表达式，用于匹配具体的模块路径，如果匹配成功就执行该规则
        test: /index\.js$/,
        // 要使用哪些 loader
        use: [
          // 配置 loader 为一个字符串
          // Webpack 执行的是会按照这个字符串加载这个文件，所以不需要我们手动的 require('./loader/test1-loader')
          './loaders/test1-loader',
          // 配置 loader 为一个对象，
          // 通过 loader 属性指定 loader 的路径，通过 options 给 loader 传递参数
          {
            loader: './loaders/test2-loader',
            options: {
              params1: "xxxx",
              params2: "xxxx"
            }
          }
        ]
      }
    ]
  }
};
```

<br/>warning
⚠️ 注意

Loader 规则是「从后往前」执行的，结合上面的案例也就说先执行 test2-loader 再执行 test1-loader！！！

<br/>

例如，我们就针对`变量 a = 1;`代码，把“变量”替换为`var`声明：

```js
module.exports = {
  mode: "development",
  module: {
    rules: [
      {
        test: /index\.js$/,
        use: [
          {
            loader: "./loaders/replace-loader",
            options: {
              changeVar: "var"
            }
          }
        ]
      }
    ]
  }
};

```

然后我们需要新建一个 loaders/replace-loader.js 的文件：

```js
module.exports = function (sourceCode) {
    console.log(sourceCode);
};
```

replace-loader.js 文件内需要使用 CommonJS 的规范导出一个函数，因为 Loader 是在 Webpack 的运行过程中被执行的，是基于 Node 环境的。

函数接受一个参数为字符串源代码，在 CLI 中输出可以看到：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694403345484-e6ef5af9-961b-4ce6-831d-ad05c9636303.png)

然后我们就可以对这个字符串进行操作了，因为我们在 webpack.config.js 中给 replace-loader 传递了参数，Loader 在运行的时候会产生一个 this 上下文对象，但是这个对象上有很多的属性不方便我们获取传递来的参数，所以我们可以解决一个工具来读取这个参数：

```bash
$ npm i -D loader-utils@1
```

```js
const loaderUtils = require("loader-utils");

module.exports = function (sourceCode) {
  console.log(sourceCode);

  const options = loaderUtils.getOptions(this);
  console.log(options);

  return sourceCode.replace("变量", options.changeVar);
};
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694403767520-a53f62bd-b242-4d67-8d5a-eb219e27ab1e.png)

当我们把“变量”替换为`var`后，再返回出去 Webpack 就能正常解析为 AST 语法树啦～

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694403927538-86156726-8039-47d2-9208-e45ff1e0e705.png)

## 🔢 Loader 处理样式
我们为什么要处理样式？

在传统的开发模块中都是通过 HTML 文件导入 CSS 文件，随着技术的更新出现了很多 css 的预处理器（用一种专门的编程语言，进行网页样式设计，然后再编译成正常的 CSS 文件）来提高我们的开发效率。

所以，我们使用构建工具的时候，不仅仅希望能对 JS 进行优雅的处理，例如把 TS 转换为 JS。同时也希望能把 CSS 文件也进行处理，例如 CSS 模块化、CSS 压缩、CSS3 自动添加前缀等！

Webpack 打包是从入口模块开始打包的，所以我们可以在入口模块中导入 CSS 文件：

```js
const style = require("./style/index.css");
console.log(style);
```

但是，`require()`一般是不能导入 CSS 文件的，只能导入 JS 文件。当 Webpack 执行的时候，它会把这段代码进行解析，然后尝试转换为 AST 语法树，可是 CSS 文件是无法被解析为 AST 的，所以就需要 Loader 来介入处理！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694404651239-aab1f261-d93a-4676-a3af-ab107284d01a.png)

例如我们对 Webpack 的配置进行更改：

```js
module.exports = {
  mode: "development",
  module: {
    rules: [
      {
        // 匹配到 .css 文件的时候执行这个规则
        test: /\.css$/,
        use: ["./loaders/css-loader"]
      }
    ]
  }
};
```

然后新建一个 loaders/css-loader.js 文件：

```js
module.exports = function (sourceCode) {
    console.log(sourceCode);
    return "";
};
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694411083564-ad42488d-8714-4277-a85a-7afaa40d95bb.png)

这样就得到了 src/style/index.css 文件的源代码。

为了能让 css 直接生效且可以被 Webpack 解析，所以我们可以使用 JS 来操作 CSS：

```js
module.exports = function (sourceCode) {
    return `let style = document.createElement("style");
    style.innerHTML = \`${sourceCode}\`;
    document.head.appendChild(style);
    module.exports = \`${sourceCode}\`;
    `;
};
```

以上代码，我们使用 JS 来动态的创建了一个`<style>`标签，然后把这个标签插入到`<head>`中去，这样就实现了样式导入！

代码的最后使用`module.exports`把源代码进行导出，这样在 src/index.js 文件中也可以拿到 src/style/index.css 的内容！

然后在 dist 目录下新建一个 HTML 文件，手动导入 dist/main.js 文件查询效果：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694411454476-21e4c1f6-8905-472d-8e3c-200e85bb906e.png)

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694411439568-750a86a1-d273-48f7-bf56-3ac27f4f1403.png)

对于一些图片资源同样可以使用 Loader 进行处理，例如`require("./imgs/template.png")`的时候，我们可以编写一个 Loader 把文件对象转换为一个 Base64 的字符串返回出去！！！

Webpack 的生态中有很多的 Loader，使用的时候只需要通过 npm 安装即可，一般情况下是不需要我们编写 Loader 的。
