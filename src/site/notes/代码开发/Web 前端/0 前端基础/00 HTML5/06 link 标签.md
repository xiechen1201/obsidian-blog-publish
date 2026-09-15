---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/00 HTML5/06 link 标签/","dg-note-properties":{}}
---

`<link />`元素用于规定当前 HTML 文档和外部资源的关系，该元素最常用的场景就是用于链接 CSS 样式，除此之外还可以链接网站的图标等等。

例如链接一个 CSS 样式：

```html
<link rel="stylesheet" href="main.css" />
```

`rel`属性表示当前文档和外部资源的关系，属性值为枚举值，常见的值有如下几个：

- `stylesheet`表示链接到的是一个 CSS 样式；

```html
<link rel="stylesheet" href="styles.css">
```

- `favicon`表示链接到的是一个网页的图标，资源的名称通常为 favicon.ico；

```html
<link rel="icon" href="favicon.ico">
```

- `preload`表示「为当前页面」提前加载指定的资源，通常用于优化性能。在页面加载的时候就获取资源，方便后续的使用；

```html
<link rel="preload" href="font.woff2" as="font" type="font/woff2">
```

`preload`也可以用来加载 JS 文件：

```html
<link rel="preload" href="main.js" as="script">
```

但是需要明白，`preload`只是提前请求并加载资源，但是不会执行资源。它的目的仅仅是提前加载，方便后续需要的时候已经加载完成，从而减少等待的时间。我们仍然需要通过`<script>`标签去加载并执行 JS 文件：

```html
<script src="main.js"></script>
```

- `prefetch`表示「为未来页面」提前加载指定的资源，不会在当前页面立即使用的资源。浏览器会在空闲的时候进行预取，以方便需要的时候资源已经加载完成。

```html
<link rel="prefetch" href="main.js" as="script">
```

和`preload`一样，都是仅加载资源但是不会执行，还需要手动的引入 JS 文件并执行。

- `manifest`表示指定一个 Web 应用程序的清单文件，帮助配置 PWA 的行为和外观；

```html
<link rel="manifest" href="/site.webmanifest">
```

site.webmanifest 的文件内容大致如下：

```json
{
  "name": "My PWA",
  "short_name": "PWA",
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

`href`属性表示外部资源的 URL，可以是相对、或者绝对路径；

`type`属性表示外部资源的 MIME 类型，通常不需要手动设置。对于样式表，默认是`text/css`；

`media`属性表示外部资源的媒体类型，常见值包括 `screen`（屏幕设备）、`print`（打印机）等。例如，只为打印机应用样式：

```html
<link rel="stylesheet" href="print.css" type="text/css" media="print">
```

`as`属性表示正在加载的内容类型，仅当设置了`rel="preload"`时，该属性为必填属性，可选枚举值[详见](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/link#%E5%B1%9E%E6%80%A7)。
