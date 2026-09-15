---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/00 HTML5/00 什么是 HTML5/","dg-note-properties":{}}
---

---
---
首先，我们要知道 HTML5 一种规范标准，不是任何的一种语言，就像 CSS 和 CSS3 的关系一样。所以 HTML5 是相对于 HTML4 的一种新的规范标准。

HTML5 是由万维网联盟（W3C）和 Web 超文本应用技术工作组（WHATWG）合作开发的，旨在改善和增强 Web 的功能，提供更好的用户体验。HTML5 引入了一系列新的元素和属性，增强了多媒体支持，改善了文档结构，并提高了性能和可访问性。

🔔 提示

要知道 HTML5 新增的内容包括标签和一些 WebAPI，而 WebAPI 是基于 JavaScript 的，它允许 Web 应用程序与浏览器和其他网络资源进行交互，例如 Geolocation API、Notification API、File API、SVG 2.0 API 等。

<br/>

主要特点和新功能：

1. 语义化标签
    1. 引入了新的语义化元素，如`<article>`, `<section>`,` <header>`, `<footer>`, `<nav>`, `<aside>`, 和`<figure>`。这些标签有助于更清晰地描述文档的结构和内容，提高了代码的可读性和搜索引擎的优化；
2. 表单增强
    1. 引入了新的表单控件和属性，如`date`, `email`, `url`, `number`, `range`, `color`等新的`<input>`类型，以及增强的表单验证功能；
3. 文档结构
    1. 新的全局属性（如`contenteditable`, `contextmenu`）和新的属性（如`placeholder`）增强了 HTML 元素的功能；
    2. 删除了一些过时的标签和属性，例如``, `<center>`, `<big>`, `<small>`等，使得 HTML 代码更加简洁和现代；
4. 多媒体支持
    1. 新增了`<audio>`和`<video>`标签，用于直接嵌入音频和视频文件，无需依赖第三方插件（如 Flash）；
3. 图形和绘图
    1. 提供了`<canvas>`元素，允许使用 JavaScript 进行 2D 绘图。这使得开发者可以创建动态的图形、游戏和数据可视化，详见：[Canvas API](https://www.yuque.com/xiechen/tggooy/zgonkf21rcd70g65)；
    2. 支持 SVG（可缩放矢量图形）以便于绘制矢量图形；
4. 离线和存储
    1. 支持 Web Storage（`localStorage`和`sessionStorage`），提供了更强大的客户端存储能力，取代了传统的`cookies`，详见：[Strong API](https://www.yuque.com/xiechen/tggooy/bzvygosklspf0t8b)；
    2. 引入了 Application Cache 和 Service Workers，用于创建离线 web 应用；
5. 增强的 JavaScript API
    1. Geolocation API：用于获取用户的地理位置；
    2. Web Workers：允许后台执行 JavaScript，提升页面性能，详见：[Web Worker API](https://www.yuque.com/xiechen/tggooy/yogc2stryc8w9kak)；
    3. Web Sockets：提供全双工通信协议，用于实时数据交换；
    4. File API：允许 web 应用处理用户的本地文件，详见：[File API](https://www.yuque.com/xiechen/tggooy/ml9ztdfzkeogfut6)；
