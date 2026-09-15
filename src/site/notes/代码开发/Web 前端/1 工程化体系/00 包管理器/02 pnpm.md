---
{"dg-publish":true,"permalink":"//web/1/00/02-pnpm/","dg-note-properties":{}}
---

pnpm 也是一种包管理器，相比 npm 和 yarn，pnpm 具有以下优势：

1、安装效率高于 npm 和 yarn

2、极其精简的 node_modules 目录

3、避免了开发时候的间接依赖问题

4、能极大的降低磁盘空间的占用

[Fast, disk space efficient package manager | pnpm](https://pnpm.io/zh/)

pnpm 和 npm 的绝大多数命令都是一致的！

安装 pnpm：

pnpm 同样也是作为 npm 包安装的。

```bash
$ npm install -g pnpm # 安装 pnpm
$ pnpm -v # 查看版本
```

如果要执行安装在本地的 CLI 命令，可以使用 pnpx，它和 npx 的功能完全一样，唯一不同的是，在使用pnpx 执行一个需要安装的命令时，会使用 pnpm 进行安装。

比如`npx mocha`执行本地的 mocha 命令时，如果`mocha`没有安装，则`npx`会自动的、临时的安装`mocha`，安装好后，自动运行`mocha`命令。

下面是 pnpm 和 npm 安装依赖后 node_modules 的目录对比：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/00%20%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8/_assets/1693387119357-d2b76596-b8ef-4e91-b984-a37a253868b0.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/00%20%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8/_assets/1693387162807-26db9ed3-9f35-454d-8681-ec0ec3e218c6.png)

可以看到，npm 由之前的嵌套安装改成现在的扁平化安装，就会导致我们明明安装的只有 mocha 一个依赖，结果 node_modules 模块却出现一堆依赖。

这个时候，如果我们开发的时候引入了一个 mocha 的依赖，也是可以正常运行的：

```js
require("chalk");
```

这样就会导致一个问题，我们的 package.json 文件根本没有这个依赖但是我们却可以正常导入。当有一天 mocha 如果不再依赖 chalk 模块的时候，我们的 node_modules 也不会有这个目录，那么就会导致运行错误，这就是间接模块导入。

而 pnpm 会把 mocha 的依赖全部放在 .pnpm 这个隐藏文件夹下面。

pnpm 的原理：

1. 同 yarn 和 npm 一样，pnpm 仍然使用缓存来保存已经安装过的包，以及使用 pnpm-lock.yaml 来记录详细的依赖版本。
2. 不同于 yarn 和 npm， pnpm 使用符号链接和硬链接（可将它们想象成快捷方式）的做法来放置依赖，从而规避了从缓存中拷贝文件的时间，使得安装和卸载的速度更快。

npm 和 yarn 的做法是一直从缓存拷贝（前提条件是缓存中存在这个包，且版本一致），这样就白白浪费了磁盘的空间。

![画板](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/00%20%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8/_assets/1724033388010-986697e9-58f4-468a-b1c5-e8d74c52c83d.jpeg)

3. 由于使用了符号链接和硬链接，pnpm 可以规避 windows 操作系统路径过长的问题，因此，它选择使用树形的依赖结果，有着几乎完美的依赖管理。也因为如此，项目中只能使用直接依赖，而不能使用间接依赖。
