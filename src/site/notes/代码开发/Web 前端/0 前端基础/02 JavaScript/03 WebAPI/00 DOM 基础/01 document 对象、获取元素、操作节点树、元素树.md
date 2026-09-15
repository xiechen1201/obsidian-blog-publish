---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/00-dom/01-document/","dg-note-properties":{}}
---

## 🔢 document 对象
`html`就是一个文档，也就是`document`。

`html`的「父节点」是`document`，`html`的「父元素」是`null`

```js
var html = document.getElementsByTagName("html")[0];
console.log(html.parentNode); //
console.log(html.parentElement);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653014180208-aa0df93c-d4a3-435e-be79-81e0b3d199b2.png)

## 🔢 获取元素
## 🔢 getElementById("IdName")

通过元素的`id`来获取元素，返回元素对象

<br/>

<br/>danger
`IE8`及以下浏览器不区分大小写

<br/>

```js
document.getElementById("testid");
```

## 🔢 getElementsByTagName("tagName")

通过元素的「标签名」来获取元素，返回元素集合伪数组

<br/>

```js
document.getElementsByTagName("p");
```

## 🔢 getElementsByClassName("className")

通过元素的「类名」来获取元素，返回元素集合伪数组

<br/>

<br/>danger
`IE8`及以下浏览器没有该方法

<br/>

```js
document.getElementsByClassName(".header");
```

## 🔢 getElementsByName("testName")

通过元素的`name`属性来获取元素，返回元素集合伪数组

<br/>

```js
document.getElementsByName("name");
```

## 🔢 querySelector("str")

以`CSS`选择器的方式来获取元素，返回元素对象

📌 `HTML5` 新引入的 `WEB API`

<br/>

```js
document.querySelector("p"); // 用标签名
document.querySelector("#id"); // 用ID
document.querySelector(".class"); // 用类名
document.querySelector(".box > .item");
```

## 🔢 querySelectorAll("str")

以`CSS`选择器的方式来获取元素，返回元素集合伪数组

📌 `HTML5` 新引入的 `WEB API`

<br/>

```js
document.querySelectorAll("div");
```

`querySelector`和`querySelectorAll`虽然好用，但是他们也有缺点：

- 性能不太好
- 不是实时更新的

下面看个案例：

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

先用`getElementsByTagName()`来试试：

```js
var lis = document.getElementsByTagName("li");
console.log(lis) // HTMLCollection(4) [li, li, li, li]
lis[1].remove()
console.log(lis) // HTMLCollection(3) [li, li, li]
```

然后再来看看`querySelectorAll()`:

```js
var lis = document.querySelectorAll("li");
console.log(lis); // NodeList(4) [li, li, li, li]
lis[1].remove();
console.log(lis); // NodeList(4) [li, li, li, li]
```

可以明显的看到`querySelectorAll()`子元素删除后打印依然是4个元素！！！

## 🔢 节点类型
🌴  要理解「节点包含元素，元素只是节点的一部分！！！」

节点的类型总共包括：

- `DOM`元素节点，代表码`1`
- 属性节点，代表码`2`
- 文本节点，代表码`3`
- 注释节点，代表码`8`
- `document`节点，代表码`9`
- `documentFragment`节点，代表码`11`

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653028457283-58416dab-b4bd-43f9-bbab-d9a7889e7d0f.png)

下面来看看如何操作节点树：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <ul>
      <li>
        <h2>我是标题标签</h2>
        <p>我是段落标签</p>
        <a href="">我是超链接</a>
        <!-- 我是注释 -->
      </li>
    </ul>
  </body>
</html>
<script>
  var p = document.getElementsByTagName("p")[0];
</script>
```

## 🔢 获取节点树
## 🔢 parentNode

查找元素的父节点

`html`的父节点是`document`，`document`的父节点是`null`

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653017449002-f0a5d501-aec5-4fde-831c-f81621f46ffe.png)

## 🔢 childNodes

查找元素的子节点

📌 该属性会返回子节点的所有类型元素，包括元素节点、文本节点、注释节点等

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653017558032-ef21ff4d-1d0c-49f1-b461-ba77d9b67627.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653017741677-b5bb65fb-78d5-48e1-98ef-9d5a597c53dd.png)

返回的多种元素节点，其中空格也属于文本节点！！！

## 🔢 firstChild

查找元素的第一个子节点

📌  返回的类型和`childNodes`一样，包括文本节点、属性节点等

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653025920189-580eff1c-f490-4842-bf3e-f1de17ed64a1.png)

## 🔢 lastChild

查找元素的最后一个子节点

📌  返回的类型和`childNodes`一样，包括文本节点、属性节点等

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653025977477-a84df498-0f66-4cad-9b62-cae857aa5a77.png)

因为`p`元素下只有`我是段落标签`这段文字，所以`firstChild`和`lastChild`返回内容一样。

## 🔢 previousSibling

查找元素的前一个节点

📌  返回的类型和`childNodes`一样，包括文本节点、属性节点等

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653018530592-7b119c9f-fe30-478c-8076-a0411d542ce0.png)

因为`p`元素的前面是空格，空格也属性节点，所以返回文本节点

## 🔢 nextSibling

查找元素的后一个节点

📌  返回的类型和`childNodes`一样，包括文本节点、文本节点等

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653018625846-0af9546d-1394-4386-a4ad-937647d921f9.png)

因为`p`元素的后面是空格，空格也属性节点，所以返回文本节点，如果`p`元素后面紧挨着`a`元素，那就就会返回`a`元素。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653018728971-a0b4f76a-a81e-44df-a26e-3b2e85035f96.png)

## 🔢 获取元素树
## 🔢 parentElement

查找元素的父元素

`html`元素的父元素是`null`

<br/>

<br/>danger
`IE8`及以下版本浏览器不支持

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653025365167-e526bae3-30e3-4bd1-8446-6e42252f1c91.png)

## 🔢 children

查找元素的子元素

📌 该属性只会返回子`DOM`元素不会返回其他类型的元素

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653025637801-129a3b30-de20-4ec0-b7af-c2fbe34a6204.png)

元素还有个`childElementCount`属性，返回子元素的数量（该数量里不包括文本和注释等）！！！

## 🔢 firstElementChild

查找元素的第一个子元素

<br/>

<br/>danger
`IE8`及以下版本浏览器不支持

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653026258000-7032c760-0097-43e2-a02a-ad9af6a3579a.png)

## 🔢 lastElementChild

查找元素的最后一个子元素

<br/>

<br/>danger
`IE9`及以下版本浏览器不支持

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653026331740-4c4592b1-bf4f-4314-8b06-382e070959e3.png)

## 🔢 previousElementSibling

查找元素的前一个元素

<br/>

<br/>danger
`IE9`及以下版本浏览器不支持

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653026456263-05740627-00dc-40d4-97b6-0fd0738fff7e.png)

## 🔢 nextElementSibling

查找元素的后一个元素

<br/>

<br/>danger
`IE9`及以下版本浏览器不支持

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653026384820-46043ef8-d36e-4a28-8b36-024e8e18727d.png)

## 🔢 是否有子节点
## 🔢 hasChildNodes()

该方法用于判断父节点下有没有子节点，包括文本、注释、元素节点等，返回`true`或者`false`

<br/>

```html
<body>
  <ul id="list"></ul>
</body>
```

```js
console.log(document.body.hasChildNodes()); // true, ul 元素在 body 下
```
