---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/08 Beacon API/","dg-note-properties":{}}
---

为了把尽量多的页面数据传递给服务端，很多的分析工具需要在页面生命周期中尽量晚的时候把数据传输到服务端。因此很多情况都是在浏览器的`unload`事件发送网络请求。

在`unload`事件触发并收集数据发送到服务端的时候会产生一个问题，因为该事件意味着页面马上就要销毁了，没有理由再发送任何结果未知的网络请求，因此`fetch()`就不适合做这个操作了。

为了解决这个问题，W3C 引入了补充性的 Beacon API。这个 API 给`navigator`对象新增了一个`sendBeacon()`方法。

这个方法接收一个 URL 和有效的数据载荷，并且会发送一个 POST 请求。如果请求成功进入了最终要发送的队列，则该方法返回`true`。

<br/>tips
有效的数据载荷有`ArrayBufferView`、`Blob`、DOMString、`FormData`实例。

<br/>

示例：

```js
// 发送 POST 请求
// URL: 'https://example.com/analytics-reporting-url'
// 请求负载：'{foo: "bar"}'

navigator.sendBeacon('https://example.com/analytics-reporting-url', '{foo: "bar"}');
```

`sendBeacon()`看起来只是一个 POST 请求的语法糖，但是具有几个重要的特性：

- 并不是只能在页面销毁的时候调用，正常情况下也可以进行调用；
- 调用成功后浏览器会把请求添加到内部的请求队列，浏览器会主动发送队列中的请求；
- 浏览器保证原始页面已经关闭的情况下也会发送请求；
- 状态码、超时和其他网络原因造成的失败是不透明的，不能通过编程的方式去处理；
- 请求时会携带调用`sendBeacon()`时所有相关的 cookie；
