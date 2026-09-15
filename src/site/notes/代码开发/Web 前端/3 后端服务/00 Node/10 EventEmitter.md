---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/3 后端服务/00 Node/10 EventEmitter/","dg-note-properties":{}}
---

---
---
`EventEmitter`是 Node 中事件管理的通用机制，很多核心的 API 都是围绕这个机制来触发事件的。

例如：

```js
const server = net.createServer((socket) => {
  socket.end('goodbye');
}).on('error', (err) => {
  // Handle errors here.
  throw err;
});
```

所有触发事件的对象都是`EventEmitter`的实例对象，实例对象使用`eventEmitter.on()`注册监听器，使用`eventEmitter.emit()`触发事件。

```js
instance.on("data", () => {});

instance.on("close", () => {});
```

下面是一个继承`EventEmitter`类并且注册和触发事件的示例：

```js
const EventEmitter = require('node:events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('event', () => {
  console.log('an event occurred!');
});

myEmitter.emit('event');
```

也可以传递参数：

```js
myEmitter.on('event', (a, b) => {
  console.log(a, b);
});

myEmitter.emit('event', 'a', 'b');
```

`EventEmitter`的内部相当于维护了多个事件队列，可以简单理解为一个普通的数组。

```js
{
    "event": [
        () => {},
        () => {}
    ]
}
```

这些事件都是同步触发的。

`EventEmitter`还有`once()`方法用于只监听一次事件，`off()`方法用于移除监听：

```js
myEmitter.once('event', () => {
  // 只触发一次
  console.log('an event occurred!');
});

myEmitter.emit('event');
myEmitter.emit('event');
```

最后，编写一个示例：创建一个 HTTP 服务器，并在处理完请求后出发一个`"response"`事件。

```js
const { EventEmitter } = require("events");
const http = require("http");

class MyRequest extends EventEmitter {
    constructor(url, options = {}) {
        super();
        this.url = url;
        this.options = options;
    }

    send(body = "") {
        const request = http.request(this.url, this.options, (res) => {
            let result = "";
            res.on("data", (chunk) => {
                result += chunk.toString("utf-8");
            });
            res.on("end", () => {
                // 触发事件
                this.emit("response", result);
            });
        });
        request.write(body);
        request.end(body);
    }
}
```

```js
// 测试调用
const request = new MyRequest("http://www.baidu.com");
request.send();
request.on("response", (res) => {
    console.log(res);
});
```

更多详见：

[Events | Node.js v23.11.0 Documentation](https://nodejs.org/docs/latest/api/events.html)
