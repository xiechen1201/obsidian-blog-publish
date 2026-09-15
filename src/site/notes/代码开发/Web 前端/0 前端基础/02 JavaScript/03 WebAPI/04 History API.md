---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/04 History API/","dg-note-properties":{}}
---

---
---
`history`对象表示当前窗口首次使用以来用户的导航历史记录。因为`history`是`window`的属性， 所以每个`window`都有自己的`history`对象。

出于安全考虑，这个对象不会暴露用户访问过的`URL`， 但可以通过它在不知道实际 URL 的情况下前进和后退。

## 🔢 go() 方法
`go()`方法可以在用户历史记录中沿任何方向导航，可以前进也可以后退。

这个方法只接收一个参数，这个参数可以是一个整数，表示前进或后退多少步。负值表示在历史记录中后退（类似点击浏览器的“后退”按钮），而正值表示在历史记录中前进（类似点击浏览器的“前进”按钮）。

```js
// 后退一页
history.go(-1);

// 前进一页
history.go(1);

// 前进两页
history.go(2);
```

<u></u>

`go()`有两个简写方法：`back()`和`forward()`。

这两个方法模拟了浏览器的后退按钮和前进按钮。

```js
// 后退一页
history.back();

// 前进一页
history.forward();
```

`history`对象还有一个`length`属性，表示历史记录中有多个条目。

这个属性反映了历史记录的数量，包括可以前进和后退的页面。对于窗口或标签页中加载的第一个页面，`history.length`等于 1。

## 🔢 hashchange 事件
现代 Web 应用程序开发中最难的环节之一就是历史记录管理。用户每次点击都会触发页面刷新的时代早已过去，“后退”和“前进”按钮对用户来说就代表“帮我切换一个状态”的历史也就随之结束了。

为解决这个问题，首先出现的是`hashchange`事件。HTML5 也为`history`对象增加了方便的状态管理特性。

`hashchange`会在页面 URL 的散列变化时被触发，开发者可以在此时执行某些操作。而状态管理`API`则可以让开发者改变浏览器`URL`而不会加载新页面。

```html
<a href="#login">login</a>
<a href="#logout">logout</a>
```

```js
window.addEventListener("hashchange", function (e) {
  console.log(e);
});
```

## 🔢 pushState() 方法
这个方法接收 3 个参数：一个`state`对象、一个新状态的标题、一个（可选的）相对URL。

```js
let stateObject = {foo:"bar"};

window.history.pushState(stateObject, "My title", "baz.html");
```

`pushState()`方法执行后，状态信息就会被推到历史记录中，浏览器地址栏也会改变以反映新的相对 `URL`。除了这些变化之外，即使`location.href`返回的是地址栏中的内容，浏览器也不会向服务器发送请求。

第一个参数应该包含正确初始化页面状态所必需的信息。为防止滥用，这个状态的对象大小是有限制的，通常在 500KB～1MB 以内。

第二个参数并未被当前实现所使用，因此既可以传一个空字符串也可以传一个短标题。

## 🔢 popstate 事件
因为`pushState()`会创建新的历史记录，所以也会相应地启用“后退”按钮。此时单击“后退” 按钮，就会触发`window`对象上的`popstate`事件。

`popstate`事件的事件对象有一个`state`属性，其中包含通过`pushState()`第一个参数传入的`state`对象：

```js
oBtn.onclick = function(){
  window.history.back();
}

window.addEventListener("popstate", (event) => {
  let state = event.state;
  if (state) {
    // 第一个页面加载时状态是 null
    processState(state);
  }
});
```

基于这个状态，应该把页面重置为状态对象所表示的状态（因为浏览器不会自动为你做这些）。记住，页面初次加载时没有状态。因此点击“后退”按钮直到返回最初页面时`event.state`会为`null`。

## 🔢 replaceState()  方法
可以通过`history.state`获取当前的状态对象，也可以使用`replaceState()`并传入与`pushState()`同样的前两个参数来更新状态。

更新状态不会创建新历史记录，只会覆盖当前状态：

```js
window.history.replaceState({newFoo: "newBar"}, "New title");
```

传给`pushState()`和`replaceState()`的`state`对象应该只包含可以被序列化的信息。因此，`DOM`元素之类并不适合放到状态对象里保存。

<br/>warning
⚠️ 注意

使用`HTML5`状态管理时，要确保通过`pushState()`创建的每个“假”`URL`背后都对应着服务器上一个真实的物理`URL`。否则，单击“刷新”按钮会导致 404 错误。所有单页应用程序（`SPA，Single Page Application`）框架都必须通过服务器或客户端的某些配置解决这个问题。

<br/>
