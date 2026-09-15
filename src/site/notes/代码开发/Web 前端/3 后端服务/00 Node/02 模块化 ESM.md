---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/3 后端服务/00 Node/02 模块化 ESM/","dg-note-properties":{}}
---

ESM 模块化是 JavaScript 代码进行打包和重复使用的官方标准，使用`import`导入模块，使用`export`导出模块。

在 Node 中如何使用 ESM 模块化呢？

- 使用 .mjs 结尾的文件；
- 在最近的 package.json 文件中添加`"type"`为`"module"`；
- 使用 CLI 的时候使用`--input-type=module`;

反过来，如果：

- 使用 .cjs 结尾的文件；
- 在最近的 package.json 文件中添加`"type"`为`"commonjs"`；
- 使用 CLI 的时候使用`--input-type=commonjs`;

都将解析为 CommonJS 模块化。

如果缺少任何一种显式地的标记，Node 会检查模块源代码查找 ES 语法，如果发现这种语法则使用 ES 模块的形式运行代码，否则以 CommonJS 的形式运行模块。

## import 导入
`import`语句的指定符是`from`关键字后面的字符串，例如：

```js
import { sep } from 'node:path';
```

指定符有三种类型：

1. 相对路径，例如`./`或者`../`;
2. 绝对路径，例如`/`;
3. 裸路径，例如直接写一个模块的名字；

和 CommonJS 一样，包内的模块文件可以在包的后面添加子路径来访问：

```js
import package from "package/path/index.js";
```

## 强制文件拓展名
使用`import`解析相对或者绝对路径的时候，必须要提供文件的拓展名，这种行为和浏览器环境保持一致：

```js
import data from "./startup/index.js";
```

## URL
ESM 还可以加载 URL 的路径，不过必须对特殊字符串进行编码，例如 # 、¥、& 等等。

默认情况下，仅支持带有`file:`、`node:`、`data:`协议的 URL，除非使用自定义的[ HTTPS 加载器](https://nodejs.org/docs/latest/api/module.html#import-from-https)，否则 Node 本身不支持`https://example.com`这样的 URL。

如果在解析同一个指定符，但是不同的查询参数的时候，则会多次加载模块：

```js
import './foo.mjs?query=1'; // 执行模块
import './foo.mjs?query=2'; // 执行模块
```

另外在导入`data:`的指定符的时候可以使用 MIME 类型：

- `text/javascript`
- `application/json`
- `application/wasm`

```js
import 'data:text/javascript,console.log("hello!");';
import _ from 'data:application/json,"world!"' with { type: 'json' };
```

以上代码中`data:`URL 允许在不依赖外部文件的情况下直接提供数据，形式：`data:[MIME类型],[数据]`，`with`用于指定导入内容的类型。

`node:`则可以加载 Node 的内置模块：

```js
import fs from 'node:fs/promises';
```

## 内置模块
内置模块提供了公共 API 的命名导出形式，也有默认导出的形式。

```js
import fs, { readFileSync } from 'node:fs';
```

## import()
CommonJS 和 ESM 都支持使用`import()`，在 CommonJS 中可以使用其导入 ESM 模块。

```js
// main.cjs

import("./math.mjs").then((data) => {
    console.log(data);
    // [Module: null prototype] { add: [Function: add] }
});
```

```js
// math.mjs

export function add(a, b) {
  return a + b;
}
```

## import.meta
该对象包含了模块的一些元信息。

## import.meta.dirname（Node v23.9.0 非标准）
返回当前模块的目录路径。

```js
console.log(import.meta.dirname);
// /Users/xiechen/Documents/code-personal/s-learn-code/nodejs/02/code
```

## import.meta.filename（Node v23.9.0 非标准）
返回当前模块的文件路径。

```js
console.log(import.meta.filename);
// // /Users/xiechen/Documents/code-personal/s-learn-code/nodejs/02/code/main.mjs
```

## import.meta.url
用于获取当前模块的完整 URL 路径。

```js
console.log(import.meta.url);
// file:///Users/xiechen/Documents/code-personal/s-learn-code/nodejs/02/code/main.mjs
```

## import.meta.resolve()（Node v23.9.0 非标准）
用于解析模块的路径，但是不加载模块。

```js
console.log(import.meta.resolve("./math.cjs"));
// file:///Users/xiechen/Documents/code-personal/s-learn-code/nodejs/02/code/math.cjs
```

## 加载 CommonJS 模块
`import`语句可以加载 ESM 和 CommonJS 模块，但是`import`只能在 ESM 中使用，但是`import()`同时支持 ESM 和 CommonJS。

在使用`import`导入 CommonJS 的时候，通过`module.exports`导出的内容将作为默认导出。

```js
// main.mjs

import math from "./math.cjs";
console.log(math);

/*
  { add: [Function (anonymous)], minus: [Function (anonymous)] }
*/
```

```js
// math.js

exports.add = (a, b) => {
  return a + b;
};

module.exports.minus = (a, b) => {
  return a - b;
}
```

对于 CommonJS 的`require()`函数，目前只能支持同步的 ESM（Node v23.9.0 非标准），也就是不支持顶层 await 的 ESM 模块。

当 ESM 加载一个 CommonJS 模块的时候，Node 会为 CommonJS模块创建一个命名空间包装器，并始终提供一个指向`module.exports`的值作为`default`的导出。

例如使用`* as name`的形式对 CommonJS 导出进行重命名的时候可以更加直观的看出模块命名空间对象：

```js
import * as math from "./math.cjs";
console.log(math);

/*
[Module: null prototype] {
  add: [Function (anonymous)],
  default: { add: [Function (anonymous)], minus: [Function (anonymous)] },
  minus: [Function (anonymous)]
}
*/
```

## ESM 和 CommonJS 的区别
1、ESM 没有`require()`、`exports`或者`module.exports`；

2、在大多数情况下，ESM 的`import`可以加载 CommonJS 模块；

3、没有`__filename`或者`__dirname`;

4、没有`require.resolve()`函数；

5、没有`require.cache`，因为 ESM 有自己独立的缓存；
