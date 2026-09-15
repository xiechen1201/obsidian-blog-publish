---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/03-storage/","dg-note-properties":{}}
---

HTML5 Web 存储是一个比 Cookie 更好的本地存储方式。

客户端存储数据的两个对象为：

LocalStorage 用于长久保存整个网站的数据，保存的数据没有过期时间，直到手动删除。

SessionStorage 用于临时保存同一窗口（或标签页）的数据，在关闭窗口或标签页之后将会删除这些数据。

## 🔢 Storage 对象
`Storage`是一个构造函数，该构造函数有以下几个原型方法：

- `getItem(name)`取得给定`name`的值；
- `setItem(name, value)`设置给定`name`的值；
- `removeItem(name)`删除给定`name`的名/值对；
- `clear()`删除所有值；
- `key(index)`取得给定数值位置的名称；

实例化对象本身还有一个`length`属性，通过该属性可以确定`Storage`对象中保存了多少名/值对。但是无法确定对象中所有数据占用的空间大小。

## 🔢 localStorage
作为在客户端持久存储数据的机制，要访问同一个`localStorage`对象，页面必须来自同一个域（子域不可以）、在相同的端口上使用相同的协议。

因为`localStorage`对象是`Storage`对象的实例对象，所以可以通过`Storage`的原型方法操作数据。

```js
// 使用方法存储数据
localStorage.setItem("name", "Nicholas");

// 使用属性存储数据
localStorage.getItem("name");
```

## 🔢 sessionStorage
`sessionStorage`对象只存储会话数据，这意味着数据只会存储到浏览器或者页面关闭。

因为`sessionStorage`对象与服务器会话紧密相关，所以在运行本地文件时不能使用。存储在`sessionStorage`对象中的数据只能由最初存储数据的页面使用，在多页应用程序中的用处有限。

<u></u>

因为`sessionStorage`对象是`Storage`的实例，所以可以通过`Storage`的原型方法操作数据。

```js
// 使用方法存储数据
sessionStorage.setItem("name", "Nicholas");

// 使用属性存储数据
sessionStorage.getItem("name");
```

## 🔢 存储事件
每当`Storage`对象发生变化时，都会在文档上触发`window`对象的`storage`事件。

这个事件有以下 4 个属性：

- `domain`：存储变化对应的域。 
- `key`：被设置或删除的键。 
- `newValue`：键被设置的新值，若键被删除则为 null。 
- `oldValue`：键变化之前的值。

```js
window.addEventListener("storage", (event) => {
  alert(`Storage changed for ${event.domain}`)
});
```

<br/>warning
⚠️ 注意

对于`sessionStorage`和`localStorage`上的任何更改都会触发`storage`事件，但`storage`事件不会区分这两者。

<br/>

## 🔢 Cookie 与 Storage 对象的区别
|  | Cookie | LocalStorage | SessionStorage |
| --- | --- | --- | --- |
| 大小 | 约 4k | 约 10M | 约 5M |
| HTML 版本 | HTML4/5 | HTML5 | HTML5 |
| 数据共享 | 所有同源窗口中都是共享的 | 所有同源窗口中都是共享的 | 不在不同的浏览器窗口中共享，即使是同一个页面 |
| 有效期 | 只在设置的过期时间之前一直有效，即使窗口或浏览器关闭，如果没有设置则会在浏览器关闭后删除 | 始终有效，窗口或浏览器关闭也一直保存，除非清除缓存或手动删除 | 仅在当前浏览器窗口关闭前有效 |
| HTTP 请求时的请求头 | 内容会随着请求一并发送到服务器 | 仅存在本地，不会与服务器发生任何交互 | 仅存在本地，不会与服务器发生任何交互 |
| 存储环境 | 服务器和浏览器 | 仅浏览器 | 仅浏览器 |
