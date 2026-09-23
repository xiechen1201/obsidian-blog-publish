---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/15 同步、异步加载 JavaScript/","dg-note-properties":{}}
---

在`HTML`中加载`JS`文件都是同步加载的，这是为了避免因页面还没有渲染完成，`JS`文件内可能存在对`DOM`操作而产生错误。

相关文章：

[时间线、解析与渲染](https://www.yuque.com/xiechen/pyimri/xsdu45)

`script`标签支持两个属性用于异步加载：

1. `defer`：异步加载`JS`文件，但是不会立即执行，<u>而是在页面渲染完成后再执行</u>
2. `async`：该属性是`W3C`的标准，是`HTML5`的属性，它和`defer`一样会异步加载`JS`文件，<u>但是加载完成后立即执行</u>

```html
<!-- 不管有多少文件都会并行下载，不会阻塞页面解析渲染 -->
<script src="./util1.js" defer="defer"></script>
<script src="./util2.js" defer></script>

<!-- 同样并行下载不阻塞解析渲染，但是一加载完成后立马执行 -->
<script src="./util4.js" async="async"></script>
<script src="./util5.js" async></script>
```

> [!danger]
>
> ⚠️ 使用异步加载`JS`文件需要注意以下几点：
>
> 1. 不对文档`DOM`直接操作的可以用异步加载
> 2. 工具类的库，完全不直接操作`DOM`的`JS`文件可以用异步加载（指的是调用的时候才生效）
> 3. 如果浏览器同时兼容`defer`和`async`且`script`上同时存在这两个属性，那么浏览器只会认`async`属性


除了以上两种方式，我们还可以手动按需加载`JS`文件：

```js
var script = document.createElement("script");
script.type = "text/javascript";
script.async = true;
script.src = "./utils.js"; // 设置 src 后会立马加载文件，但是不会执行
document.body.appendChild(script); // 这个时候才会执行 js 文件
```

但是这样的方式会阻塞`window.onload`事件的触发，因为该事件是在所有资源加载完成后才会触发，这样的方式仍然属于加载。

那么我们可以在`window.onload`事件触发后再异步加载`JS`文件：

```js
(function () {
  function asyncLoad() {
    var s = document.createElement("script");
    s.type = "text/javascript";
    s.async = true;
    s.src = "./utils.js"; // 设置src后会立马加载文件，但是不会执行
    document.body.appendChild(s);
  }

  window.onload = function () {
    asyncLoad();
  };
})();
```
