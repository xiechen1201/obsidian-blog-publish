---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/17 Draggable API/","dg-note-properties":{}}
---

---
---
## 🔢 可拖动能力
默认情况下，图片、链接和文本是可拖动的，无需任何的代码就可以拖动。如果要让其他元素也可以进行拖动，那么就必须给元素设置一个 HTML5 的属性`draggable`属性，表示是否可以进行拖动。

```html
<div draggable="true">...</div>
```

## 🔢 拖放事件
Draggable 事件的关键是要分清楚每个事件在哪里触发，有的事件发生在「被拖动元素」上，有的事件则发生在「放置目标」上。

在某个元素被拖动的时候，会按顺序执行以下事件：

- `dragstart`
- `drag`
- `dragend`

示例：

```html
<div style="border: 1px solid red; width: 100px; height: 100px"
  draggable="true"
  id="draggableBox"></div>
```

```js
const draggableBox = document.getElementById("draggableBox");
function handleEvent(e){
  console.log("🚀 ~ handleEvent ~ e:", e.type)
}

draggableBox.addEventListener("dragstart", handleEvent);
draggableBox.addEventListener("drag", handleEvent);
draggableBox.addEventListener("dragend", handleEvent);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776326790157-c709e8a8-0e5a-4d57-9359-0bcd618d1c7a.png)

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776326800052-7b485b31-85d1-403e-9d5a-756a5af1c816.png)

在鼠标未松开的情况下，`drag`事件持续触发，当松开手指鼠标的时候触发`dragend`事件。

在元素被拖动的时候，浏览器不会更改被拖动元素的外观，而是让开发者更改其外观。

当把拖动元素「拖动到」一个目标元素上，则目标元素会触发：

- `dragenter`
- `dragover`
- `dragleave`或`drop`

示例：

```html
<div
      style="border: 1px solid #333; width: 500px; height: 500px"
      id="targetBox"></div>
```

```js
const targetBox = document.getElementById("targetBox");
function targetEvent(e) {
  console.log("🚀 ~ targetEvent ~ e:", e.type);
}
targetBox.addEventListener("dragenter", targetEvent);
targetBox.addEventListener("dragover", targetEvent);
targetBox.addEventListener("dragleave", targetEvent);
targetBox.addEventListener("drop", targetEvent);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776327381768-0c70a3d4-9efb-48c8-a163-65bf5cb19d65.png)

只要被拖动到元素拖动到目标元素上，`dragenter`事件会立即触发，接着会触发`dragover`事件，并且被拖动元素在目标元素上持续放置，那么`dragover`事件也会触发触发。如果被拖动元素被拖到目标元素外，则触发`dragleave`事件。

如果被拖动元素放在了目标元素上，则会触发`drop`事件，而不是`dragleave`事件。

<br/>warning
⚠️ 注意

要触发`drop`事件有个前提，详见下一小节。

<br/>

## 🔢 自定义放置目标
默认情况下，一个`<div>`元素不会被视作为是一个「有效的放置目标」。

<br/>tips
默认有效的放置目标：

- `<input />`或`<textarea>`
- 设置了`contenteditable`属性的可编辑元素

<br/>

如果把元素拖动到一个非有效的目标放置元素上，那么这些元素是不允许放置的，无论用户怎么操作都不会触发`drop`事件。

不过，可以通过覆盖`dragenter`和`dragover`的默认事件来把任何元素转化为有效的放置元素。

对事件监听进行改造：

```js
const targetBox = document.getElementById("targetBox");
function targetEvent(e) {
  console.log("🚀 ~ targetEvent ~ e:", e.type);
}
function targetEnterOverEvent(e) {
  e.preventDefault();
  console.log("🚀 ~ targetEnterOverEvent ~ e:", e.type);
}
targetBox.addEventListener("dragenter", targetEnterOverEvent);
targetBox.addEventListener("dragover", targetEnterOverEvent);
targetBox.addEventListener("dragleave", targetEvent);
targetBox.addEventListener("drop", targetEvent);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776328766873-b6580777-c436-4c1d-9620-4f220e2bb46a.png)

以上代码新增了一个`targetEnterOverEvent`方法，专门用于覆盖`dragenter`和`dragover`事件的默认行为，这样就成功触发了`drop`事件。

## 🔢 dataTransfer 对象
为了实现再拖放过程中数据的传递，HTML5 将`dataTransfer`对象纳入了标准，用于从被拖动元素到目标元素传递字符串数据。

`dataTransfer`对象有两个主要的方法：`getData()`和`setData()`，分别用于获取值和设置值。

`setData()`的第一个参数和`getData()`的唯一参数表示要设置的数据类型：`"text"`或`"URL"`。

```js
// 传递文本
event.dataTransfer.setData("text", "some text");
let text = event.dataTransfer.getData("text");

// 传递 URL
event.dataTransfer.setData("URL", "http://www.wrox.com/");
let url = event.dataTransfer.getData("URL");
```

当从文本框拖动文本的时候，浏览器会自动调用`setData()`以`"text"`的形式把文本存储。类似的，当拖动图片或者超链接的时候，也会调用`setData()`以`"URL"`的形式把链接存储。

```html
 <a href="http://www.baidu.com" >这是一个测试内容</a>
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776340151325-032db5e4-5fb2-46a2-82a7-8a9a8b05512e.png)
