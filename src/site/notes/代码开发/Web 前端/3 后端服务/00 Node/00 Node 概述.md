---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/3 后端服务/00 Node/00 Node 概述/","dg-note-properties":{}}
---

什么是 Node？

Node 是一个基于 Chrome V8 引擎的 JS 运行环境，他比浏览器拥有更多的能力。

[Node.js — Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs)

> JavaScript 一般指在浏览器运行的 JS，NodeJS 指能在 Node 环境运行的 JS。
>

## 浏览器中的 JavaScript
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716329722032-3fc980e3-45c0-4ef1-ac4b-ba6eb6202cdf.png)

在浏览器中 JavaScript 主要包括 ECMAScript 和 WebAPI 两部分，ESMAScript 是 JS 的核心内容，这一部分也能运行在 NodeJS 中，而 WebAPI 主要是浏览器提供的一些接口，这些 API 的能力非常的有限。在 Node 环境中不存在这些 API 也就无法调用了，不过在 Node 环境中 Node 也提供了 Node 的 API 让我们可以操作电脑系统。

## NodeJS 的 JavaScript
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716329942863-922ca0ea-c79a-44e2-90c7-f95c956f6d7a.png)

到了 NodeJS 环境，主要使用的就是 NodeAPI，这些 API 比 WebAPI 更加的强大，可以操作电脑系统。

## 分层结构对比
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716330027245-c7d8506a-ab36-4d6b-92e3-491795e64f5c.png)

浏览器提供了有限的 Web API 能力，JS 只能使用浏览器提供的功能做「有限」的事情。

Node 提供了完整控制计算机的能力，几乎可以通过 NodeAPI 实现「对整个操作系统」的控制，例如操作文件、操作网卡，操作数据库等。

## Node 的优点
- 单线程的应用程序，使用异步回调模式，不存在线程之间的的竞争，可以高效的处理高并发请求；
- 采用事件驱动和非阻塞 I/O 模型，这使得它非常适合处理大量并发连接，以及 I/O 密集型的应用；
- Node 可以跨平台运行，包括 Window、MacOS、Linux；
- ...

## 使用 Node 可以干什么？
- 开发桌面程序；
- 开发服务器程序；
    - ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716330555795-7936e2d6-cbd1-46a2-976b-26ac263c22fc.png)
    - 这样的结构通常应用在微型站点上，客户端发起请求，服务端的 Node 服务器监听请求并完成请求的处理、响应、和数据库的交互；
    - ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716330615801-55c08850-181f-4ceb-97e7-c02c7d0635e3.png)
    - 这样的模式更加的常见，Node 作为一个中间层，不用做太多的事情，不做和业务相关的事情，只是简单的转发请求，但可能会做一些额外的功能，例如：简单信息的记录（日志、用户偏好、广告信息）、静态资源的托管、缓存...

## Node 对前端的影响
Node 的出现给前端带来一次重大的变革，从今以后前端开发：

- 全栈的 JavaScript，可以使用同一种语言编写前端和后端应用程序；
- 推动模块化的发展，促进了前端的工程化和组件化的开发；
- 工具链的革新，出现了一些基于 Node 环境的前端构建工具，例如 Webpack、Gulp、Vite 等等，极大地改善了前端的开发流程；
- 跨平台应用，借助 Node 的能力可以编译为一些跨平台的应用程序，例如 Electron、React Native 等；
