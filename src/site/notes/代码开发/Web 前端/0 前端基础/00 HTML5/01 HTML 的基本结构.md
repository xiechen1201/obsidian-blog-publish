---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/00 HTML5/01 HTML 的基本结构/","dg-note-properties":{}}
---

---
---
HTML 是由 W3C 组织定义的语言标准，HTML 是用于描述页面结构的语言。也就是页面中有什么东西，这个东西具有什么含义。

一个基本的 HTML 结构：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="keywords" content="关键字" />
    <meta name="description" content="页面描述内容" />
    <title>网页标题</title>
  </head>
  <body> </body>
</html>
```

- `<!DOCTYPE>`：用于声明指定 HTML 文档的版本为 HTML5 版本，如果不声明，浏览器可能会用兼容模式渲染，导致页面显示不一致。声明标准模式后，浏览器会按照 W3C 的标准渲染页面。
- `<html>`：表示文档根元素，`lang="en"`表示文档语言为英语；
- `<head>`：主要包含网页的基本信息和配置，包括字符集、标题、描述等，这一部分的内容不会显示到页面中；
    - `<meta>`：用于设置网页的元信息数据；
    - `<title>`：用于设置网页的标题；
- `<body>`：网页的主体内容展示区域；

浏览器中运行的网页实际上都是 .html 结尾的文件，而负责解析 HTML 和 CSS 的工作主要是由浏览器的内核负责。

浏览器主要有两部分组成：

- 外壳 Shell；
- 内核 Core（JS 引擎、渲染引擎）；

以下是五大浏览器的内核：

- Chrome：webkit/blink （webkit 是谷歌和苹果合作的成果，后来谷歌把内核独立 blink）；
- Safari：webkit；
- Firefox：gecko；
- IE：trident；
- Opera：presto；
