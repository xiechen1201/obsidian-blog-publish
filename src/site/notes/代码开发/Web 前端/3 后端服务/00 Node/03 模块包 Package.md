---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/3 后端服务/00 Node/03 模块包 Package/","dg-note-properties":{}}
---

## 包的入口
在软件包的 package.json 文件中，有两个字段可以定义软件包的入口：`main`和`exports`。这两个字段同时适用于 ESM 模块和 CommonJS 模块。

对于`main`字段，所有的 Node 版本都支持它，但是功能比较受限，仅仅可以定义包的入口。

`exports`是`main`的替代方案，可以定义多个入口，还支持不同环境，不同条件的入口解析。

当一个包中同时出现`main`和`exports`字段，则`exports`的优先级更高。

```json
{
  "name": "my-package",
  "exports": {
    ".": "./lib/index.js",
    "./lib": "./lib/index.js",
    "./lib/index": "./lib/index.js",
    "./lib/index.js": "./lib/index.js",
    "./feature": "./feature/index.js",
    "./feature/index": "./feature/index.js",
    "./feature/index.js": "./feature/index.js",
    "./package.json": "./package.json"
  }
}
```

当使用`exports`定义入口后，可以阻止用户使用未定义的入口，例如我直接加载的是`src`下面的文件：

```js
const core = require("my-package/src/index.js");

/*
  Error [ERR_PACKAGE_PATH_NOT_EXPORTED]: Package subpath './src/index.js' is not defined by "exports" in xxx
*/
```

另外，`exports`导出也可以选择使用带或不带子路径的整个文件夹：

```json
{
  "name": "my-package",
  "exports": {
    ".": "./lib/index.js",
    "./lib": "./lib/index.js",
    "./lib/*": "./lib/*.js",
    "./lib/*.js": "./lib/*.js",
    "./feature": "./feature/index.js",
    "./feature/*": "./feature/*.js",
    "./feature/*.js": "./feature/*.js",
    "./package.json": "./package.json"
  }
}
```

## 子路径
使用`exports`字段时，可以把主入口点写成`"."`，同时自定义其他子路径，从而控制模块的导入方式。

```json
{
  "exports": {
    ".": "./index.js",
    "./submodule.js": "./src/submodule.js"
  }
}
```

之后，包的使用者只能导入包的名称或者`submodule.js`子路径：

```js
import packageName from "package-name";
import packageName from "package-name/submodule.js";
```

导入其他子路径就会抛出 ERR_PACKAGE_PATH_NOT_EXPORTED  的错误。

如果整个包只有一个`.`主入口，就可以将`exports`的值简写成字符串的形式：

```js
{
  "exports": {
    ".": "./index.js"
  }
}
// 简写 ==>
{
  "exports": "./index.js"
}
```

## 有条件出口
条件导出提供了根据特定条件映射到不同路径的方法，CommonJS 和 ESM 都支持条件导出。

```json
{
  "exports": {
    "import": "./index-module.js",
    "require": "./index-require.cjs"
  },
  "type": "module"
}
```

Node 实现了以下条件，这些条件按照从最具体到最不具体的规则排序：

- `node-addons`，模块需要被编译为 Node 原生插件（例如使用 C++ 编写的 .node 文件）；
- `node`，无论是加载普通 JavaScript 模块还是原生插件；
- `import`，当使用`import`或`import()`加载；
- `require`，当使用`require()`加载；
- `module-sync`，无论是`import`或`import()`还是`require()`加载；
- `default`，始终匹配的通用回退（也适用于浏览器）；

在`exports`中，KEY 的顺序非常重要，在条件匹配的过程中，较早的项优先于后面的项。

有条件出口也适用于子路径：

```json
{
  "exports": {
    ".": "./index.js",
    "./feature.js": {
      "node": "./feature-node.js",
      "default": "./feature.js"
    }
  }
}
```

另外条件出口还支持再嵌套条件出口，例如，要定义一个只在 Node 中使用双模块入口点而不在浏览器中使用双模入口点的软件包：

```json
{
  "exports": {
    "node": {
      "import": "./feature-node.mjs",
      "require": "./feature-node.cjs"
    },
    "default": "./feature.mjs"
  }
}
```

## 社区定义条件
Node 默认只支持：

- `node-addons`（原生插件）
- `node`（Node 环境）
- `import`（ES 模块加载）
- `require`（CommonJS 加载）
- `module-sync`（同步模块）
- `default`（兜底默认值）

由于自定义的软件包需要明确定义才能确保正确使用，因此社区整理了几个已知常见的软件包条件：

- `types`：给 TypeScript 用的，用来指定类型定义文件（必须放在第一位）；
- `browser`：浏览器环境专用的代码入口；
- `development`：开发模式专用，比如带调试信息的代码（和 `production` 不能共存）；
- `production`：生产环境专用，比如优化后的代码（和 `development` 不能共存）；

## 子路径导入
除了`exports`字段之外，还有一个`imports`字段用于创建仅适用于包内导入的「私有映射」。

`imports`字段的 KEY 必须是以`#`开头的，以确保它们是和外部的软件包区分开的。

```json
{
  "imports": {
    "#dep": "./dep-polyfill.js"
  },
  "dependencies": {
    "dep-node-native": "^1.0.0"
  }
}
```

```js
import "#dep";
// ==> 直接加载 ./dep-polyfill.js 模块
```

当然，子路径导入也支持使用`*`批量导入：

```json
{
  "exports": {
    "./features/*.js": "./src/features/*.js"
  },
  "imports": {
    "#internal/*.js": "./src/internal/*.js"
  }
}
```

```js
import internalZ from '#internal/z.js';
```

## 其他字段
详见：[https://nodejs.org/docs/latest/api/packages.html#nodejs-packagejson-field-definitions](https://nodejs.org/docs/latest/api/packages.html#nodejs-packagejson-field-definitions)
