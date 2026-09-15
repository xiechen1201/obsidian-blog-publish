---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/00 包管理器/00 npm/04 发布 npm 包/","dg-note-properties":{}}
---

准备工作：

1、国内的开发者有的时候可能会使用淘宝的镜像源作为 npm 包的下载地址，所以我们要删除 npm 的 registry 属性，让其使用默认的 registry 地址。

```bash
$ npm config delete registry
```

2、登录 npm 的官网，然后注册账户

3、本机使用 CLI 登录刚才注册的账户

```bash
# 登录账户
$ npm login

# 查看当前已登录的账户
$ npm whoami

# 登出账号
$ npm logout
```

4、创建工程目录（注意不能和 npm 上现有的包重名，所以起名前最好先去 npm 上搜一下有没有这个包）

5、使用`npm init`命令初始化包，编写 package.json 文件的相关信息

如果是要创建一个范围包，则需要使用`npm init --scope@username`

6、编写工程代码

<br/>tips
编写包的时候，如果依赖了其他的 npm 包，则必须正确的安装到 package.json 文件的`dependencies`或`devDependencies`中。

<br/>

7、创建并编写 README 文件

发布：

1、开发代码

2、确认好本次要发布的版本号

3、发布 npm 包

```bash
$ npm publish

# 如果是范围包，并且想要发布为公开，则需要运行
$ npm publish --access public
```

默认情况下，运行`npm publish`后将使用`latest`作为包的标记，也就是表示包的最新版本。如果想要添加不同的标签，可以运行：

```bash
$ npm publish --tag <tag>

# 示例
$ npm publish --tag beta
```

如果想要给包的某个版本添加标签，可以运行：

```bash
$ npm dist-tag add <package-name>@<version> [<tag>]
```

4、发布成功后，登录 npm 官网，查看是否已经发布成功

5、新建一个工程目录，安装我们刚才发布的包，测试使用

```bash
$ npm install 包名
```
