---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/00 DOM 基础/02 节点属性、封装方法、DOM 结构/","dg-note-properties":{}}
---

---
---
## 🔢 节点属性
## 🔢 nodeName

`node.nodeName`返回节点的名字

该属性是「只读」属性

元素节点访问该属性返回的名字是大写的

字符串的方法：

`str.toLowerCase()` 字符串转小写

`str.toUpperCase()`字符串转大写

<br/>

```html
<div class="box" id="box" style="background-color: green">
  我是文本节点
  <!--我是注释 -->
  <h1>我是h1</h1>
  <a href="">我是超链接</a>
  <p>我是段落标签</p>
</div>
```

```js
console.log(document.nodeName); // #document
console.log(document.getElementsByTagName("div")[0].nodeName); // DIV，元素节点返回的字符串为大写
console.log(document.getElementsByTagName("div")[0].firstChild.nodeName); // #text
console.log(document.getElementsByTagName("div")[0].childNodes[1].nodeName); // #commment
console.log(document.getElementsByTagName("div")[0].childNodes[3].nodeName); // H1
```

## 🔢 nodeValue

`node.nodeValue`返回节点的值

该属性是「只读」属性

只有文本节点、注释节点、属性节点才有该属性

<br/>

```js
console.log(document.getElementsByTagName("div")[0].nodeValue); // null
console.log(document.getElementsByTagName("div")[0].firstChild.nodeValue); // "  我是文本节点  "
console.log(document.getElementsByTagName("div")[0].childNodes[1].nodeValue); // 我是注释
console.log(document.getElementsByTagName("div")[0].getAttributeNode("id").nodeValue); // box，getAttributeNode 获取属性节点(了解即可)
```

## 🔢 nodeType

`node.nodeType`返回节点的类型值

该属性是「只读」属性

之前学习过节点的类型总共包括：

- `DOM`元素节点，代表码`1`
- 属性节点，代表码`2`
- 文本节点，代表码`3`
- 注释节点，代表码`8`
- `document`节点，代表码`9`
- `documentFragment`节点，代表码`11`

<br/>

```js
console.log(document.getElementsByTagName("div")[0].nodeType); // 1
console.log(document.getElementsByTagName("div")[0].firstChild.nodeType); // 3
console.log(document.getElementsByTagName("div")[0].childNodes[1].nodeType); // 8
```

因为`childNodes`返回是节点类型的集合，里面包含了所有的节点类型，我们可以利用`nodeType`封装一个方法只获取「元素节点」。

```js
function elementChildren(node) {
  var arr = [];
  var children = node.childNodes;
  for (var i = 0; i < children.length; i++) {
    if (children[i].nodeType === 1) {
      arr.push(children[i]);
    }
  }
  return arr;
}

let res = elementChildren(document.getElementsByTagName("div")[0]);
console.log(res); // [h1, a, p]
```

## 🔢 attributes

返回节点的属性集合

该属性「可读可写」

<br/>

```js
console.log(document.getElementsByTagName("div")[0].attributes); // NamedNodeMap {0: class, 1: id, 2: style, class: class, id: id, style: style, length: 3}
console.log(document.getElementsByTagName("div")[0].attributes[0]); // class=“box”
console.log(document.getElementsByTagName("div")[0].attributes[0].nodeValue); // box
```

## 🔢 tagName

返回元素节点的大写标签名

<br/>

```js
document.getElementsByTagName("div")[0].tagName; // DIV
```

## 🔢 DOM 结构
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1653293339456-8fdd6ba0-808f-4990-a1f0-fed0faaf06da.png)

`DOM`结构树分为多种对象，它们之间是一种原型链的继承关系。

TODO！！！

```plain
## document 对象
document.__proto__ => HTMLDocument.prototype
HTMLDocument.prototype.__proto__ => Document.prototype
Document.prototype.__proto__ => Node.prototype
所以 getElementById、getElementsByTagName 等方法都继承 Document.prototype

## CharacterData 对象
Text.prototype.__proto__ => CharacterData.prototype
Comment.prototype.__proto__ => CharacterData.prototype

## Element
HTMLElement.prototype.__proto_ => Element.prototype
XMLElement.prototype.__proto_ => Element.prototype
```

## 🔢 获取全部标签
`document.getElementsByTagName()`方法可以接受通配符`*`来获取文档全部的标签。

```js
console.log(document.getElementsByTagName("*"));
// HTMLCollection(13) [html, head, meta, meta, meta, title, body, span, div#box.box, h1, a, p, script, viewport: meta, box: div#box.box]
```

## 🔢 获取文档信息
获取文档的`head`

```js
console.log(document.head);
```

获取文档的`body`

```js
console.log(document.body);
```

获取文档的`title`

```js
console.log(document.title);
```

获取文档的`html`

```js
console.log(document.documentElement);
```
