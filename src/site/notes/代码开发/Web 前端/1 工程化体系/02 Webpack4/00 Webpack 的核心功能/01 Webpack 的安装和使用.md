---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/02 Webpack4/00 Webpack 的核心功能/01 Webpack 的安装和使用/","dg-note-properties":{}}
---

---
---
Webpack 能帮助开发者把开发时候写的很爽的代码「转换为」运行时很爽的代码，它是基于模块化（包括又不限于 CommonJS、ESModule）的打包（构建）工具，它把一切视为模块。

## 🔢 Webpack 的简介
Webpack 的大致工作流程：

1、从入口文件开始分析

2、分析依赖关系（包括 CommonJS、ESModule 等）

3、把源代码进行解析、压缩、合并等一系列操作

4、最终生成运行时态代码，也就是可以在浏览器上运行的代码

Webpack 的特点：

- 「为前端工程化而生」：Webpack 致力于解决前端工程化，特别是浏览器端工程化中遇到的问题，让开发者集中注意力编写业务代码，而把工程化过程中的问题全部交给 Webpack 来处理。
- 「简单易用」：支持零配置，可以不用写任何一行额外的代码就使用 Webpack。
- 「强大的生态」：Webpack 是非常灵活、可以扩展的，Webpack 本身的功能并不多，但它提供了一些可以扩展其功能的机制，使得一些第三方库可以融于到 Webpack 中。
- 「基于 Node」：由于 Webpack 在构建的过程中需要读取文件，因此它是运行在 Node 环境中的。
- 「基于模块化」：Webpack 在构建过程中要分析依赖关系的方式就是通过模块化导入语句进行分析的，它支持各种模块化标准，包括但不限于 CommonJS、ES6 Module，最后打包处理的代码既不是 CommonJS 也不是 ESModule，而是 Webpack 自己实现的模块化函数。

## 🔢 Webpack 安装
要使用 Webpack 需要安装以下两个依赖：

```bash
$ npm install -D webpack@4
$ npm install -D webpack-cli@3
```

可以不安装指定的版本，写这篇文章的时候 Webpack5 已经上线，但是和相关的工具存在诸多的兼容问题，所以本篇文章还是安装 Webpack4 来学习。

- webpack：核心包，提供了相关的 API
- webpack-cli：运行 Webpack 的 CLI 命令，可以调用 Webpack 的 API 来完成构建

当然，也可以通过全局的方式安装 Webpack，但这不是不被推荐的，因为我们的项目可能使用 Webpack 的版本不一致。

然后，cd（进入）到我们的工程目录中运行 Webpack：

```bash
$ cd ./demo
$ npx webpack
```

这个时候，Webpack 默认就会找当前目录下的 ./src/index.js 作为入口文件，然后进行分析打包，最后把编译的结果输出到 ./dist/main.js 文件中去。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1693896756855-569aa9c0-8353-4bfd-9aae-16ffe0209e96.png)

我们可以还可以通过`--mode`参数来指定开发环境，这样 Webpack 执行的流程就会有所区别：

```bash
$ npx webpack --mode=development
$ npx webpack --mode=production
```
