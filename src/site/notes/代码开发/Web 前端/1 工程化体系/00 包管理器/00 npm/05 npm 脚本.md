---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/1 工程化体系/00 包管理器/00 npm/05 npm 脚本/","dg-note-properties":{}}
---

我们在开发的时候会反复的使用很多的 CLI 命令，比如启动本地服务器、打包、测试、代码格式化等等。

这些命令太多的时候会导致难以记忆。

于是，npm 非常贴心的支持了脚本，只需要在 package.json 中配置 scripts 字段，即可配置各种执行文件的脚本命令，最后我们通过`npm run 脚本名称`运行脚本即可。

例如：

```json
// 其他的配置文件
"script":{
  "start": "node ./index.js",
  "dev": "webpack-dev-server --config ./build/webpack.config.js",
  "build": "webpack --config ./build/webpack.config.js"
}
```

最后，运行：

```bash
# $ ./node_modules/.bin/webpack-dev-server --config ./build/webpack.config.js
# $ npx webpack-dev-server --config ./build/webpack.config.js

$ npm run start
$ npm run dev
$ npm run build
```

这样我们就不用找到 ./node_modules/.bin 下的执行文件了。

不仅如此，npm 还对某些常用的脚本名称进行了简化，下面的脚本名称是不需要使用 run 的：

- start
- stop
- test

```bash
$ npm start
$ npm stop
$ npm test
```

script 还可以配置任何电脑可运行的 CLI 命令：

```json
"scripts": {
  "ls": "ls -l"
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/00%20%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8/00%20npm/_assets/1693211285639-3a226d91-bdc4-471a-8dd6-515c644a5c9a.png)
