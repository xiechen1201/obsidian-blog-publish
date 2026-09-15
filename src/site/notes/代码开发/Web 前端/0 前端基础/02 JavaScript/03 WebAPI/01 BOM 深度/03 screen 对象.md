---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/01-bom/03-screen/","dg-note-properties":{}}
---

`window.screen`表示浏览器窗口外面的客户端显示器的信息。

因大部分属性只支持`IE`浏览器，所以不常用。

```js
window.screen;

// window 也可以省略
screen;
```

部分属性：

| 属 性 | 说明 |
| --- | --- |
| `orientation` | 返回`ScreenOrientationAPI`中屏幕的朝向 |
