---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/09 EventSource API/","dg-note-properties":{}}
---

EventSource API 是和 WebSocket API 功能很类似的一个 API，其支持客户端通过一个 HTTP 连接接收服务器的实时更新。

> [!tip]
> WebSocket 更强大和灵活，因为它是全双工通道，可以双向通信。
>
>SSE 是单向通道，只能服务器向浏览器发送，因为流信息本质上就是下载。如果浏览器向服务器发送信息，就变成了另一次 HTTP 请求。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776827909537-bead0cb9-3fff-40ce-ada7-a96e69d071cb.png)

要使用 EventSource API 服务端必须用 SSE 的格式向客户端发送事件。SSE 是一种轻量的协议，用于通过 HTTP 从服务器向客户端发送基于文本的事件。每个事件都包含一个字段名和值，由`:`分隔。多个事件使用两个换行符分隔。

客户端使用`EventSource`对象监听事件，并处理连接和解析事件。

```js
const eventSource = new EventSource("https://api.example.com/events");

// 链接建立成功后触发
eventSource.open = eveny = {}

// 监听事件
eventSource.onmessage = event => {
  console.log(event.data);
};

// 监听错误
eventSource.onerror = error => {
  console.log("Error:", error);
};

// 关闭连接
eventSource.close()
```

## 服务端实现
服务端向客户端发送数据，必须是 UTF-8 的编码，且具有以下的响应头：

```http
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: hello

data: world

data: { "msg": "hi" }
```

例如原生 Node 实现：

```js
const http = require("http");

http.createServer((req, res) => {
  if (req.url === "/sse") {
    res.writeHead(200, {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      "Connection": "keep-alive",
      "Access-Control-Allow-Origin": "*"
    });

    let count = 0;

    setInterval(() => {
      count++;

      // 👇 核心：不断往 response 写数据
      res.write(`data: message ${count}\n\n`);
    }, 1000);
  }
}).listen(3000);
```

除了`data`，还可以写：

```markdown
# 表示自定义的事件类型，默认是message事件
event: customEvent
data: hello

# 相当于每一条数据的编号,浏览器用 lastEventId 属性读取这个值
id: 123
data: message

# 指定浏览器重新发起连接的时间间隔
retry: 3000
```

前端监听：

```js
eventSource.addEventListener("customEvent", (e) => {
  console.log(e.data);
});
```
