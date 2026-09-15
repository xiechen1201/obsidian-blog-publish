---
{"dg-publish":true,"permalink":"//web/1/02-webpack4/04/10-e-slint/","dg-note-properties":{}}
---

ESlint 是一个代码风格检查工具，当我们书写的代码不满足 ESlint 规则的时候它就会抛出警告或错误。

[检测并修复 JavaScript 代码中的问题。 - ESLint - 插件化的 JavaScript 代码检查工具](https://zh-hans.eslint.org/)

如果是团队开发代码，一人一个代码风格就会产生冲突混乱，所以就需要使用工具进行统一。

ESlint 不会影响代码的执行，只是会对代码的书写风格进行检查，所以本文和 Webpack 的关系不大！

## 🔢 使用
ESlint 通常配合编辑器使用：

1、在 VSCode 中安装 ESlint 的插件

该插件会自动读取工程目录中 ESlint 的配置文件，然后根据配置文件去读取工程目录的代码，给予开发中警告或者错误。

ESlint 也能结合构建工具进行使用，但是构建工具只能在构建的时候进行提示，而编辑器会立马进行提示。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/04%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/_assets/1698996769413-4d592397-fed1-470b-bb51-31136c97dc26.png)

虽然编辑器中代码飘红，但是不影响功能正常执行。

2、安装 ESlint

```bash
$ npm init @eslint/config
```

然后根据问题一步步的创建 ESlint 的配置。

3、创建配置文件

默认情况下，你操作完第二走就会自动给你创建一个配置文件，如果没有你可以执行下面的操作。

```bash
$ npx eslint --init
```

## 🔢 配置
例如下面是我默认生成的配置文件：

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: 'airbnb-base',
  overrides: [
    {
      env: {
        node: true,
      },
      files: ['.eslintrc.{js,cjs}'],
      parserOptions: {
        sourceType: 'script',
      },
    },
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  rules: {},
};
```

## 🔢 env
改属性配置的是代码的执行环境。

- browser：代码是否在浏览器环境中运行；
- es6：是否启用 ES6 的全局 API，例如 Promise 等；

## 🔢 parserOptions
该属性配置指定 eslint 对哪些语法的支持。

- ecmaVersion: 支持的 ES 语法版本
- sourceType
    - script：传统脚本
    - module：模块化脚本

## 🔢 parser
ESlint 的工作原理是先将代码进行解析，然后按照规则进行分析。

ESlint 默认使用 Espree 作为其解析器，你可以在配置文件中指定一个不同的解析器，例如对 React、TypeScript 进行单独的解析器解析。

## 🔢 globals
改属性用于配置可以使用的额外的全局变量。

```js
module.exports = {
  "globals": {
    "var1": "readonly",
    "var2": "writable"
  }
}
```

```js
// 编辑器不会飘红
console.log(var1);
console.log(var1);
```

ESlint 支持注释形式的配置，在代码中使用下面的注释也可以完成配置：

```js
/* global var1, var2 */
/* global var3:writable, var4:writable */
```

## 🔢 extends
改属性配置要继承哪些库的配置，值为字符串或者数组。

继承一些已经配置好的规则，不需要自己手动一个一个的进行配置

```js
{
  "extends": "eslint:recommended"
}
```

## 🔢 ignorePatterns
改属性配置忽略对哪些目录和文件的检查。

或者新建一个 .eslintignore 文件，他们遵循同样的语法。

```js
{
    "ignorePatterns": ["temp.js", "/vendor/*.js"],
    "rules": {
        //...
    }
}
```

详见：

[忽略文件 - ESLint - 插件化的 JavaScript 代码检查工具](https://zh-hans.eslint.org/docs/latest/use/configure/ignore)

## 🔢 rules
该属性配置 ESlint 的规则集。

每条规则影响某个方面的代码风格。

详见：

[规则参考 - ESLint - 插件化的 JavaScript 代码检查工具](https://zh-hans.eslint.org/docs/latest/rules/)

每条规则都有下面几个取值：

- off 或 0 或 false: 关闭该规则的检查
- warn 或 1 或 true：警告，不会导致程序退出
- error 或 2：错误，当被触发的时候，程序会退出

除了在配置文件中使用规则外，还可以在注释中使用：

```js
/* eslint eqeqeq: "off", curly: "error" */
```
