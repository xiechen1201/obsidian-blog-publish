---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/16 Clipboard API/","dg-note-properties":{}}
---

Clipboard API 的目的是为了取代旧的`document.execCommand()`操作剪贴板。

## 权限
犹豫剪贴板涉及用户的隐私，因此通常都需要用户进行授权才可以进行使用。

剪贴板权限可以分为两部分：

- 读取权限`'clipboard-read'`
- 写入权限`'clipboard-write'`

以下代码展示了如何判断用户是否进行了授权：

```js
navigator.permissions.query({name: "clipboard-read"}).then(result => {
  if (result.state === "granted") {
    // 用户授权了读的权限
  }
});
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776481612093-87438354-0c9c-4ae0-a9a2-3d980b27868a.png)

> [!tip]
>
> `result.state`的返回值：
>
> - `granted`权限已允许，可以直接使用对应能力；
> - `denied`权限已拒绝，调用相关能力通常会失败；
> - `prompt`还没有明确允许或拒绝，真正使用能力时可能弹出授权提示；
>
> `navigator.permissions.query`不会让页面展示授权的弹窗，只有调用 Clipboard API 的时候才会展示。


另外浏览器只允许页面在激活状态下使用 Clipboard API，如果页面没有获得焦点的时候进行读取、写入剪贴板，那么浏览器会抛出`“DOMException: Document is not focused.”`的错误。

如果想要在用户没有任何操作的时候进行剪贴板的读取，则必须添加一个`allowWithoutGesture`属性，让用户授权在没有任何操作的时候读取剪贴板的内容。

```js
navigator.permissions.query({
  name: "clipboard-read",
  allowWithoutGesture: true
}).then(({ state }) => console.log(state));
```

## 读写文本
文本是剪贴板最常见的格式，Clipboard API 提供了`readText()`和`writeText()`方法来进行操作文本的读取和写入。

```js
navigator.clipboard.readText().then((clipText) => {
  console.log(clipText);
});

navigator.clipboard.writeText("Put this in the clipboard").then(() => {
  console.log('Writing to clipboard was successful!');
});
```

## 剪贴板事件
当用户通过设备进行剪切、复制或者粘贴的时候，我们可以监听`cut`、`copy`和`paste`事件来执行某些操作。

这些事件都会冒泡，因此可以绑定到`document`对象上进行监听：

```js
document.addEventListener("copy", async () => {
  console.log("Copied text:", await navigator.clipboard.readText()); });
```

## 处理非文本数据
对于非文本数据，Clipboard API 提供了`read()`和`write()`方法。这两个方法是处理数据的通用方法，需要使用 ClipboardItem 来组织数据数组，返回一个状态的 Promise。

ClipboardItem 是一个构造函数，接收一个数据对象作为参数，对象使用 MIME 类型作为 Key，实际的值作为 Value。

> [!tip]
>
> ClipboardItem 允许传入多种对象，支持多种数据类型。例如一个同一份内容的时候既可以表示为纯文本，也可以表示为 HTML 两种格式。这样用户就可以支持 HTML 的应用中粘贴为 HTML，否则粘贴为纯文本。


例如给 ClipboardItem 传入一个图片 Blob：

```js
document.querySelector("#read").addEventListener("click", async () => {
  // 创建一个 canvas 对象并转化为 blob 对象
  const blob = await new Promise((resolve) =>
    document.createElement("canvas").toBlob(resolve, "image/png")
  );

  // 创建一个剪贴板对象
  const clipboardItem = new ClipboardItem({
    "image/png": blob
  });

  // 以数组的形式写入剪贴板
  navigator.clipboard.write([clipboardItem]);
});
```

如果要读取剪贴板中的数据，则需要迭代 ClipboardItem 的数组，匹配每种 MIME 类型，然后再读取内容：

```js
document.querySelector("#read").addEventListener("click", async () => {
  // clipboard.read() 返回 ClipboardItem 对象的数组
  for (let clipboardItem of await navigator.clipboard.read()){
    // 查询图片类型
    if(clipboardItem.types.includes("image/png")){
      // 使用 getType 来访问剪贴板数据
      const oImage = await clipboardItem.getType("image/png");

      const img = document.createElement("img");
      img.src = URL.createObjectURL(oImage);
      document.body.appendChild(img);
    }
  }
})
```
