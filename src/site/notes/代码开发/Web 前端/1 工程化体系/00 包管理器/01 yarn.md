---
{"dg-publish":true,"permalink":"//web/1/00/01-yarn/","dg-note-properties":{}}
---

## 🔢 简介
yarn 是一个全新的包管理工具，但是它依然使用的是 npm 的 registry 源。yarn 提供了全新的命令对包进行管理。

为什么会出现 yarn 呢？这是因为 npm 之前出现的很多问题：

1、依赖目录嵌套太深，windows 系统无法支持太深的目录。

例如模块 A 依赖 B，B 又依赖 C，C 依赖 D 等等。

```markdown
node_modules
	A
    	node_modules
      	  B
        	    node_modules
          	      c
            	        node_modules
              	          ...
```

2、下载速度慢。

同样是因为嵌套的关系，npm 下载包的时候只能串行下载，导致宽带资源没有完全利用。

多个相同版本的包会被重复下载。

3、控制台输出的信息太复杂

4、工程拷贝问题。

由于 npm 的版本依赖可以是模糊的，可能会导致工程拷贝移植后，依赖的确切版本不一致。

针对上述问题，yarn 进行了优化：

1、使用扁平化的目录结构

```markdown
node_modules
	A
  B
  C
  D
```

2、并行下载

3、使用本地缓存

4、控制台信息精简

5、使用 yarn-lock 文件记录依赖的关系

yarn 还优化了下面的内容：

1、增加功能强大的命令

2、让命令更加语义化

3、本地安装的 CLI 可以直接使用 yarn 进行启动

4、将全局安装的目录当作一个普通的工程，生成 package.json 文件，便于全局安装移植

yarn 给 npm 的市场带来巨大的压力，于是 npm 学习了 yarn 的理念也进行了改进：

1、目录扁平化

2、并行下载

3、本地缓存

4、使用 package-lock 记录确切依赖

5、增加了大量的命令别名

6、内置了 npx，可以启动本地的 CLI 工具

7、极大的简化了控制台输出

总之，npm6 之后的版本和 yarn 已经非常接近了。

## 🔢 yarn 常用命令
1、全局安装 yarn

```bash
$ npm install --global yarn
```

yarn 是作为 npm 的一个包，所以依然需要使用 npm 进行安装，之后就可以使用 yarn 的命令了

2、初始化

```bash
$ yarn init [--yes/-y]
```

3、安装包

```bash
$ yarn [global] add 包名
$ yarn [global] add 包名 [--dev/-D]
$ yarn [global] add 包名 [--exact/-E]
```

4、根据 package.json 文件安装全部的依赖

```bash
$ yarn install [--production/--prod]
```

5、运行脚本或本地的 CLI（node_modules/.bin）

```bash
$ yarn run 脚本名
$ yarn run CLI名
```

脚本名为 start、stop、test 时可以省略 run

6、查询 node_modules 的 .bin 目录

```bash
$ yarn [global] bin
```

7、查看包信息

```bash
$ yarn info 包名 [子字段]
```

8、查看已经安装的依赖

```bash
$ yarn [global] list [--depth=依赖深度]
```

`yarn list`命令和`npm list`不同，yarn 输出的信息更加丰富，包括顶级目录结构、每个包的依赖版本号。

9、查询需要更新的包

```bash
$ yarn outdated
```

10、更新包

```bash
$ yarn [global] upgrade [包名]
```

11、卸载包

```bash
$ yarn remove 包名
```

12、验证 package.json 和 yarn.lock 文件记录是否一致

```bash
$ yarn check
```

该命令对防止篡改非常有用。

13、查看本地安装的包哪些有已知漏洞

```bash
$ yarn audit
```

以表格的形式列出，漏洞级别分为以下几种：

- INFO：信息级别
- LOW: 低级别
- MODERATE：中级别
- HIGH：高级别
- CRITICAL：关键级别

14、查看为什么安装某个包，哪些包用到了这个包

```bash
$ yarn why
```

15、通过脚手架快速创建项目

React 或者 Vue 会提供一些脚手架，所谓脚手架，就是使用一个命令来搭建一个工程结构。

过去，我们都是使用如下的做法：

1. 全局安装脚手架工具
2. 使用全局命令搭建脚手架

由于大部分脚手架工具都是以 create-xxx 的方式命名的，比如 react 的官方脚手架名称为 create-react-app

因此，可以使用`yarn create`命令来一步完成安装和搭建。

例如：

```bash
$ yarn create react-app my-app
# 等同于下面的两条命令
$ yarn global add create-react-app
$ create-react-app my-app
```
