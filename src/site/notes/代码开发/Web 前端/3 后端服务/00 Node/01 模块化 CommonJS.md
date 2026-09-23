---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/3 后端服务/00 Node/01 模块化 CommonJS/","dg-note-properties":{}}
---

目前 Node（Node v23.9.0）已经完全支持 CommonJS 和 ESM 两种模块化标准，基于此写几篇文章学习一下用法和差异。

本文先来介绍传统的 CommonJS 模块化。

CommonJS 说 Node 最原始的模块化方案，在 Node 中每个单独的「文件」都可以被看作是一个「模块」。

```js
// foo.js

// 加载 circle 模块
const circle = require('./circle.js');
console.log(`The area of a circle of radius 4 is ${circle.area(4)}`);
```

```js
// circle.js
const { PI } = Math;

// 导出 area 和 circumference 两个函数
exports.area = (r) => PI * r ** 2;
exports.circumference = (r) => 2 * PI * r;
```

模块中使用`require()`导入一个模块的内容，使用`module.exports`或`exports`导出模块的内容。

`module.exports`和`exports`其实是一个对象，`exports`是`module.exports`的简写形式。因此，如果将`exports`赋值为一个新的对象，则会丢失对`module.export`的指向：

```js
exports.a = 1;
module.exports.b = 2;

console.log(exports); // { a: 1, b: 2 }
console.log(module.exports); // { a: 1, b: 2 }

// 重新赋值 exports
exports = { c: 3 };

console.log(exports); // { c: 3 } （这个 exports 只是局部变量）
console.log(module.exports); // { a: 1, b: 2 } （module.exports 没有改变）
```

模块内的变量都是私有的，因为 Node 将模块的代码封装在一个函数中，例如下面的形式：

```js
(function (exports, require, module, __filename, __dirname) {
  // 模块的原始代码在这里运行
});
```

默认情况下，Node 将符合以下条件的文件视为 CommonJS 模块：

- 以 .cjs 结尾的文件；
- 以 .js 结尾的文件，且最近的父级目录的 package.json 中`"type"`为`"commonjs"`；
- 以 .js 结尾的文件，且最近的父级目录的 package.json 中没有`"type"`字段；
- 以 .js 结尾的文件，且任何父级目录都没有 package.json 文件；
- 文件的代码可以评估为 CommonJS；

通常情况，即使所有文件模块都是 CommonJS 也应该在 package.json 文件中设置`"type"`为`"commonjs"`。

## 如何使用加载 ESM（非标准）？
> [!warning]
>
> 写这篇文章的时候参考的是 Node v23.9.0 的文档，这个版本加载 ESM 的方式已经成为了 1.2 候选发布版本。
>
> 关于新特性的版本，详见：[https://nodejs.org/docs/latest/api/documentation.html#stability-index](https://nodejs.org/docs/latest/api/documentation.html#stability-index)


`require()`仅支持符合下面要求的 ESM 模块：

- 以 .mjs 结尾的文件；
- 以 .js 结尾的文件，且最近的父级目录的 package.json 中`"type"`为`"module"`；
- 以 .js 结尾的文件，且最近的父级目录的 package.json 中没有`"type"`字段，且代码可以被评估为 ESM；

如果加载的模块符合上面的要求，就可以被视为是一个 ESM 模块。`require()`加载这个模块的时候会返回命名空间对象，这种情况下动态的`import()`类似，只不过是同步运行并直接返回命名空间对象。

```js
// main.js

const distance = require("./distance.mjs");
console.log(distance);
/*
    [Module: null prototype] {
        distance: [Function: distance]
    }
*/
```

```js
// distance.mjs

export function distance(a, b) {
  return Math.sqrt((b.x - a.x) ** 2 + (b.y - a.y) ** 2);
}
```

如果一个 ESM 是默认导出，则会在命名空间对象上添加一个`__esModule: true`的属性。

```js
// main.js

const point = require("./point.mjs");
console.log(point);
/*
    [Module: null prototype] {
        __esModule: true,
        default: [class Point]
    }
*/
```

```js
// point.mjs

export default class Point {
  constructor(x, y) { this.x = x; this.y = y; }
}
```

如果 ESM 模块同时包含命名导出和默认导出的时候，默认导出的结果会放在命名空间对象的`default`属性上。

```js
// main.js

const point = require("./point.mjs");
console.log(point);

/*
  [Module: null prototype] {
    __esModule: true,
    default: [class Point],
    distance: [Function: distance]
  }
*/
```

```js
// point.mjs
export default class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
}

export function distance(p1, p2) {
    return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
}
```

如果要定义`require()`直接返回模块的内容，而不是命名空间对象，ESM 可以使用`"module.exports"`来输出内容。

```js
// main.js

const point = require("./point.mjs");
console.log(point);

/*
  [class Point]
*/
```

```js
// point.mjs

export default class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
}

export function distance(p1, p2) {
    return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
}

export { Point as 'module.exports' }
```

不过，使用`"module.exports"`导出内容后，CommonJS 将无法访问命名导出，因此可以在默认导出上添加一个属性作为命名导出：

```diff
// point.mjs

export default class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
++    static distance = distance;
}

export function distance(p1, p2) {
    return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
}

export { Point as 'module.exports' }
```

如果`require()`加载的模块包含顶层的`await`，则会抛出 `ERR_REQUIRE_ASYNC_MODULE`的错误，这种情况下应该使用`import()`去加载 ESM。

## 缓存
模块在首次加载后会被缓存，当其他模块再次加载这个模块的时候，会直接从缓存中获取。

只要不修改`require.cache`对象中的内容，模块就会一直被缓存。

```js
// main.js

require("foo");
require("foo");
require("foo");
require("foo");

// foo.js 文件之后执行一次
```

```js
console.log(require.cache);

/*
    [Object: null prototype] {
        '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo2/main.js': {
            ...
        },
        '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo2/foo.js': {
            ...
        }
    }
*/
```

如果想让模块多次执行，可以导出一个函数，并在需要的时候调用这个函数。

```js
// main.js

const runModule = require("./foo");

runModule();
runModule();
```

```js
// foo.js

module.exports = function () {
  console.log("模块被执行了！");
};
```

> [!warning]
>
> 模块的名称是区分大小写的，因此`require("foo")`和`require("FOO")`是两个不同的模块。


## 内置模块
Node 提供了非常多的内置模块，内置模块是 Node 本身就提供的模块，不需要通过包管理器安装就可以直接加载的模块。

内置模块可以使用`node:`作为前缀进行标识，这种情况下会绕过`require()`的缓存。另外即使存在和内置模块同名的模块文件，也会被优先加载。

```js
require("node:fs");
```

所有的内置模块都可以通过`module.builtinModules`获取：

```js
console.log(require('node:module').builtinModules);

/*
    [
    '_http_agent',         '_http_client',        '_http_common',
    '_http_incoming',      '_http_outgoing',      '_http_server',
    '_stream_duplex',      '_stream_passthrough', '_stream_readable',
    '_stream_transform',   '_stream_wrap',        '_stream_writable',
    '_tls_common',         '_tls_wrap',           'assert',
    'assert/strict',       'async_hooks',         'buffer',
    'child_process',       'cluster',             'console',
    'constants',           'crypto',              'dgram',
    'diagnostics_channel', 'dns',                 'dns/promises',
    'domain',              'events',              'fs',
    'fs/promises',         'http',                'http2',
    'https',               'inspector',           'inspector/promises',
    'module',              'net',                 'os',
    'path',                'path/posix',          'path/win32',
    'perf_hooks',          'process',             'punycode',
    'querystring',         'readline',            'readline/promises',
    'repl',                'stream',              'stream/consumers',
    'stream/promises',     'stream/web',          'string_decoder',
    'sys',                 'timers',              'timers/promises',
    'tls',                 'trace_events',        'tty',
    'url',                 'util',                'util/types',
    'v8',                  'vm',                  'wasi',
    'worker_threads',      'zlib'
    ]
*/
```

## 循环调用
当模块之间存在循环引用的时候，模块在返回时可能尚未执行完成。

```js
// a.js

console.log('a starting');

exports.done = false;
const b = require('./b.js');

console.log('in a, b.done = %j', b.done);

exports.done = true;

console.log('a done');
```

```js
// b.js

console.log('b starting');

exports.done = false;
const a = require('./a.js');

console.log('in b, a.done = %j', a.done);

exports.done = true;

console.log('b done');
```

```js
// main.js

console.log('main starting');

const a = require('./a.js');
const b = require('./b.js');

console.log('in main, a.done = %j, b.done = %j', a.done, b.done);
```

当 main.js 加载 a.js 时，a.js 又加载 b.js 。此时，b.js 会尝试加载 a.js 。

为了防止出现无限循环，a.js 导出对象的未完成副本将返回给 b.js 模块。然后，b.js 将完成加载，并将其 `exports`对象提供给 a.js 模块。

所以执行结果如下：

```plain
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done = true, b.done = true
```

## 模块解析
再使用`require()`加载一个没有后缀名的模块的时候，Node 会尝试添加文件的拓展名：.js、.json、.node，因此如果想要加载非这些拓展名的文件的时候，一定要写全拓展名。

```js
require('./file.cjs')
```

另外，以`/`开头的路径是模块文件的绝对路径:

```js
require('/home/marco/foo.js');
```

以`./`开头的路径是模文件的相对路径：

```js
require('./circle');
```

如果不是`/`或`./`与`../`开头的文件，则会到`node_modules`目录中查找：

```js
require("lodash");
```

## 文件夹作为模块
如果想要将一个文件夹作为模块，则该文件夹下面应该创建一个 package.json 文件，并拥有`main`字段指定模块的入口：

```json
{
  "name" : "some-library",
  "main" : "./lib/some-library.js"
}
```

当外部加载这个模块的时候，就会根据`main`字段作为整个文件夹模块的入口：

```js
require('./some-library');

// ==> ./some-library/lib/some-library.js
```

如果文件夹中没有 package.json 文件或者 package.json 文件没有`main`字段，那么 Node 就会尝试加载目录中的 index.js 或者 index.node 文件作为入口。

```js
require('./some-library');

// ./some-library/index.js
// ./some-library/index.node
```

否则，Node 将提示找不到模块。

## 从 node_modules 中加载
上面说了当使用`require()`加载模块的时候，如果不是以`/`或`./`与`../`开头的路径，则会到`node_modules`目录中查找，这里还需要补充一个：同时不是 Node 的内置模块。

例如：

```js
// foo.js

require("lodash")
```

上面的代码中加载了一个`lodash`模块：

- 该模块不是以`/`或`./`与`../`开头的路径;
- 该模块不是 Node 的内置模块；

那么这个时候就会去 node_modules 目录中加载：

- 例如是在`/home/ry/projects/foo.js`文件加载的`lodash`模块；
- Node 首先会从当前目录开始查找，并添加`/node_modules`的路径：`/home/ry/projects/node_modules/lodash`;
- 如果找不到，则向上级目录查找：`/home/ry/node_modules/lodash`；
- 如果还是找不到，依旧向上级目录查找：
    - `/home/node_modules/lodash`
    - `/node_modules/lodash`
- 直到查找到根目录；

也可以在模块的后面添加后缀路径，表示加载的是模块的特定目录或子模块：

```js
// foo.js

require("lodash/filter.js")
```

后缀路径的查找规则和上面的规则一致。

## 模块封装器
在本文开头的时候就提到了，Node 在执行模块文件之前会使用一个函数包装器对模块代码进行包装，看起来像（并不是真实实现）：

```js
(function(exports, require, module, __filename, __dirname) {
  // Module code actually lives in here
});
```

通过这样封装，Node 可以：

- 定义的变量（`var`、`let`和`const`）都是模块局部变量，不会污染全局对象；
- 提供了一些看起来是全局变量的对象，实际上是模块局部变量：
    - `module`和`exports`；
    - `__filename`和`__dirname`；

下面就来介绍一下这些变量的作用。

## __dirname
返回当前模块的目录绝对路径，等同于`path.dirname()`方法。

```js
console.log(__dirname);
// /Users/mjr

console.log(path.dirname(__filename));
// /Users/mjr
```

## __filename
返回当前模块的文件绝对路径。

```js
console.log(__filename);
// /Users/mjr/example.js
```

## exports
对`module.exports`对象的引用，书写更方便，用于暴露模块的内容。

```js
function add(num1, num2){
  return num1 + num2;
}

exports.add = add;
```

## require()
用于导入模块、JSON 和本地文件。模块的路径可以是`./`与`../`的相对路径，也可以是`/`绝对路径，或者模块名开头的裸路径。

```js
const myLocalModule = require('./path/myLocalModule');
const jsonData = require('./path/filename.json');
const crypto = require('node:crypto');
```

## require.cache
返回模块被加载后的缓存数据，如果某个模块从这个对象中被删除，那么下一次再加载这个模块的时候将会重新执行这个模块。

```js
console.log(require.cache);

/*
    [Object: null prototype] {
        '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo2/main.js': {
            ...
        },
        '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo2/foo.js': {
            ...
        }
    }
*/
```

## require.main
返回 Node 进程启动后所执行的入口模块，如果程序的入口不是 CommonJS 模块，则返回`undefined`。

```js
console.log(require.main);

/*
  {
    id: '.',
    path: '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4',
    exports: {},
    filename: '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4/main.js',
    loaded: false,
    children: [],
    paths: [
      ...
    ],
    [Symbol(kIsMainSymbol)]: true,
    [Symbol(kIsCachedByESMLoader)]: false,
    [Symbol(kIsExecuting)]: true
  }
*/
```

## require.resolve()
用于解析（查找）模块的位置，但是不加载模块，只返回解析后的模块路径。

参数：

- request： 要解析的模块路径；
- options：可选，选项；
    - paths：字符串数组，指定查找模块的路径列表。如果设置了该选项 Node 会优先从这些路径中查找模块；

示例：

```js
const localPath = require.resolve('./my-module');
console.log(localPath); // /project/src/my-module.js
```

```js
const path = require.resolve('my-package', {
  paths: [path.join(__dirname, 'src')]
});
console.log(path); // /project/src/node_modules/my-package/index.js
```

## require.resolve.paths()
返回 Node 在解析指定模块时会搜索的路径数组，帮助理解模块查找逻辑。

```js
// main.js

console.log(require.resolve.paths("lodash"));

/*
[
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/node_modules',
  '/Users/xiechen/Documents/code-personal/node_modules',
  '/Users/xiechen/Documents/node_modules',
  '/Users/xiechen/node_modules',
  '/Users/node_modules',
  '/node_modules',
  '/Users/xiechen/.node_modules',
  '/Users/xiechen/.node_libraries',
  '/Users/xiechen/.nvm/versions/node/v20.18.0/lib/node'
]
*/
```

## module
该对象指向当前模块，里面包含一些模块的信息。

```js
// main.js

console.log(module);

/*
{
  id: '.',
  path: '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4',
  exports: {},
  filename: '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4/main.js',
  loaded: false,
  children: [],
  paths: [
    ...
  ],
  [Symbol(kIsMainSymbol)]: true,
  [Symbol(kIsCachedByESMLoader)]: false,
  [Symbol(kIsExecuting)]: true
}
*/
```

## module.children
返回当前模块加载的子模块。

```js
// main.js

require("./math");

console.log(module.children);

/*
[
  {
    id: '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4/math.js',
    path: '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4',
    exports: {},
    filename: '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4/math.js',
    loaded: true,
    children: [],
    paths: [
      ...
    ],
    [Symbol(kIsMainSymbol)]: false,
    [Symbol(kIsCachedByESMLoader)]: false,
    [Symbol(kIsExecuting)]: false
  }
]
*/
```

## module.exports
用于导出模块内的数据。

```js
module.exports = {
    add: function (a, b) {
        return a + b;
    },
    sub: function (a, b) {
        return a - b;
    }
}
```

> [!warning]
>
> 不能将`module.exports`放在任何回调中。


```js
// x.js

setTimeout(() => {
  module.exports = { a: 'hello' };
}, 0);
```

```js
// y.js

const x = require("./x");
console.log(x.a); // undefined
```

## exports
是`module.exports`的快捷方式。

```js
module.exports.f = xxx
// 等同于
exports.f = xxx
```

不过，不能直接给`exports`赋值为一个新的对象，否则会丢失对`module.exports`的指向。

```js
// math.js
exports.add = function add() {};
module.exports.minus = function minus() {};

// main.js
require("./math");
// { add: [Function: add], minus: [Function: minus] }
```

```js
// math.js
exports = {a:1,b:2};
module.exports.minus = function minus() {};

// main.js
require("./math");
// { minus: [Function: minus] }
```

```js
// math.js
exports.add = function add() {};
module.exports = { a: 1, b: 2 };

// main.js
require("./math");
// { a: 1, b: 2 }
```

通过下面这个伪代码，可以更好的理解`module.exports`和`expoers`的关系：

```js
function require(/* ... */) {
  const module = { exports: {} };

  ((module, exports) => {
    // 模块内的代码
    exports.add = function add() {};
    module.exports.minus = function minus() {};
  })(module, module.exports);

  return module.exports;
}
```

## module.filename
返回模块解析后的文件路径。

```js
console.log(module.filename);

// /Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4/main.js
```

## module.id
返回模块的标识符，通常情况下是解析后的路径。

```js
// main.js

console.log(module.id); // .
// main.js 是入口文件，所以返回 .
```

## module.isPreloading
返回模块是否在 Node 预加载阶段运行。

## module.loaded
返回模块是否已经完成加载，或者正在加载。

## module.path
返回模块的目录路径。

```js
console.log(module.path);
// /Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4
```

## module.paths
返回 Node 在查找模块时会检查的路径列表。

```js
console.log(module.paths);

/*
[
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/demo4/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/code/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/01/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/nodejs/node_modules',
  '/Users/xiechen/Documents/code-personal/s-learn-code/node_modules',
  '/Users/xiechen/Documents/code-personal/node_modules',
  '/Users/xiechen/Documents/node_modules',
  '/Users/xiechen/node_modules',
  '/Users/node_modules',
  '/node_modules'
]
*/
```
