---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/10 WebSocket API/","dg-note-properties":{}}
---

WebSocket 的目标是通过一个长时连接实现和服务器的全双工、双向的通信。

| 模式 | 特点 |
| --- | --- |
| 单工 | 只能单向（比如广播） |
| 半双工 | 双向，但不能同时（像对讲机） |
| 全双工 | 双向 + 可以同时发送 |

在 JS 中创建 WebSocket 的时候，一个 HTTP 请求会发送到服务器进行初始化连接，服务器响应后，连接使用 HTTP 的`Upgrade`头部从 HTTP 协议切换到 WebSocket 协议。

也就是说WebSocket 不能通过标准的 HTTP 服务器实现，而必须使用支持该协议的专有服务器。

因为 WebSocket 使用了自定义协议，所以 URL 也不能使用 `http://`或`https://` 的协议，而是使用`ws://`或`wss://`。

## 创建使用
要使用 WebSocket 需要先进行实例化对象并传入连接：

```js
let socket = new WebSocket("ws://www.example.com/server.php");
```

> [!warning]
>
> 传递给`WebSocket`的连接必须是一个绝对的 URL。
>
> 另外，同源策略不适用于 WebSocket，所以可以访问任意站点的数据（如果服务器有响应）。


浏览器会在初始化`WebSocket`对象之后立即创建连接。该对象也有一个状态属性`readyState`用来查看当前状态，取值如下：

- `WebSocket.OPENING`，0，连接正在建立；
- `WebSocket.OPEN`，1，连接已经建立；
- `WebSocket.CLOSING`，2，连接已经关闭；
- `WebSocket.CLOSE`，3，连接已经关闭；

`WebSocket`对象没有`readystatechange`事件，而是有上述不同状态对应的其他事件。

在上述的任何阶段都可以调用`.close()`方法关闭 WebSocket 的连接。

## 发送和接收数据
在创建 WebSocket 连接后可以通过连接发送和接收数据。

要发送数据需要调用`.send()`方法并传入一个字符串、ArrayBuffer 或者 Blob。示例：

```js
let socket = new WebSocket("ws://www.example.com/server.php");

let stringData = "Hello world!";
let arrayBufferData = Uint8Array.from(['f', 'o', 'o']);
let blobData = new Blob(['f', 'o', 'o']);

socket.send(stringData);
socket.send(arrayBufferData.buffer);
socket.send(blobData);
```

当服务端向客户端发送信息的时候，Websocket 对象会触发`message`事件。

```js
socket.onmessage = function(event) {
  let data = event.data;
  // 对数据执行某些操作
};
```

## 其他事件
Websocket 对象在连接生命周期中可能触发 3 个其他事件：

- `open`，连接成功建的时候触发；
- `error`，在发生错误的时候触发，连接无法存续；
- `close`，在连接关闭的时候触发；

WebSocket 不支持`addEventListener`绑定事件，只能通过`onxxx`的方式来绑定。

```js
let socket = new WebSocket("ws://www.example.com/server.php");

socket.onopen = function() {
  alert("Connection established.");
};

socket.onerror = function() {
  alert("Connection error.");
};

socket.onclose = function() {
  alert("Connection closed.");
};
```

以上三个事件，只有`error`事件对象有额外的信息：

- `wasClean`，布尔值，表示连接是否干净的关闭；
- `code`来自服务器的状态码；
- `reason`包含来自服务器的消息；
