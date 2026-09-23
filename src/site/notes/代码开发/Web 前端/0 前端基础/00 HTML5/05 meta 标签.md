---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/00 HTML5/05 meta 标签/","dg-note-properties":{}}
---

什么是`<meta />`标签？

该标签是 HTML 中用于提供关于 HTML 文档（或称为网页）的元数据标签。元数据并不会直接显示在网页中，但是对浏览器、搜索引擎和其他网络服务非常的重要。

`<meta />`标签位于`<head>`标签之中，用于设置当前网页的编码、描述、关键字、作者信息、视口设置、重定向等信息。可以说，`<meta />`标签对网页具有至关重要的作用。

> [!tip]
> 元数据：也就是数据的数据，结合本段，元数据就是 HTML 文档的数据，记录的是对 HTML 文档的信息描述。

例如使用`<meta />`标签设置当前网页的字符编码：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body></body>
</html>
```

下面我们将根据`<meta />`标签的作用进行分类描述：

1、字符编码的设置。

```html
<meta charset="UTF-8">
```

`charset`属性用于指定 HTML 文档的字符编码，确保页面的内容可以正常显示。不同的浏览器或用户的系统默认字符编码可能不同，如果不明确指定编码，浏览器可能会选择错误的编码方式，导致页面显示乱码。

2、页面的描述和关键词。

```html
<meta name="description" content="这是一个介绍 HTML 基础的网页。">
<meta name="keywords" content="HTML, meta标签, 网页设计">
```

description 和 keywords 对 SEO 非常的重要，有助于搜索引擎理解页面内容并在搜索结果中展示描述。

3、设置视口。

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

viewport 用于控制页面在移动设备上的显示和缩放行为，使页面在手机、平板等设备上有良好的显示效果。

4、网页作者和版权。

```html
<meta name="author" content="John Doe">
<meta name="copyright" content="© 2024 Company Name">
```

5、设置 HTTP 响应头。

refresh 用于设置刷行和重定向：

```html
<meta http-equiv="refresh" content="30"> <!-- 每30秒刷新页面 -->
<meta http-equiv="refresh" content="5; url=https://example.com"> <!-- 5秒后重定向到其他页面 -->
```

cache-control 和 pragma 用于控制网页的缓存：

```html
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="pragma" content="no-cache">
```

Content-Language 用于设置页面的语言：

```html
<meta http-equiv="Content-Language" content="zh-CN">
```
