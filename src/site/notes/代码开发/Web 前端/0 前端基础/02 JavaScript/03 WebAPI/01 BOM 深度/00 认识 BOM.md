---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/01 BOM 深度/00 认识 BOM/","dg-note-properties":{}}
---

我们都知道`JavaScript`是由：`ECMAScript`、`DOM`和`BOM`三大部分组成的。

而`BOM`表示`Browser Object Model`浏览器对象模型。

`BOM`是针对浏览器相关交互的方法和接口的合集

通俗的话：`BOM`让`JS`和浏览器进行对话，获取浏览器信息和操作浏览器。

`BOM`的核心是`window`对象

1、`window`对象表示浏览器窗口

2、所有 JS 全局对象、函数、变量（包括`document`）都是`window`的对象成员

```js
window.document.getElementById("#id")
```

`BOM`不像其他两个有规范：

`ECMAScript`：通过`ECMA-262`标准化的脚步程序设计语言

`DOM`: `W3C`

`BOM`: 没有规范（浏览器厂商对其功能定义不相同，所以兼容性非常不好）

`BOM`的组成

`window`：`window`对象上直接定义的属性和方法

`Navigator`：浏览器信息

`History`：浏览器当前窗口访问的历史纪录

`Location`：获取当前页面的地址信息、页面重定向等

`Screen`：浏览器屏幕的相关信息

`Frames`：框架相关的信息获取和操作
