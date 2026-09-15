---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/00 DOM 基础/05 innerHTML 拓展篇/","dg-note-properties":{}}
---

---
---
## 🔢 innerHTML/outerHTML
`innerHTML`表示设置或者获取元素的`HTML`，另外还有个`outerHTML`属性表示替换掉包含父元素的所有内容。

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
    <div class="box">
      <li>这是列表</li>
      <p>这是段落</p>
    </div>
  </body>
</html>
```

先看看`innerHTML`的效果：

```js
var oBox = document.getElementsByClassName("box")[0];
oBox.innerHTML = "<span>这是span</span>"
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1655272459024-9d6f17b5-bd8a-4096-8c07-6750f7630fef.png)

再看看`outerHTML`的效果：

```js
var oBox = document.getElementsByClassName("box")[0];
oBox.outerHTML = "<h1>这是span</h1>";
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1655272515077-a04f0def-8c88-4e9d-b720-1e6e6074353c.png)

可以看到`outerHTML`把父节点`div`也给替换了！！！

## 🔢 innerHTML 的执行过程
`innerHTML`的操作过程是比较消费性能的，每次设置`innerHTML`都会经过如下的步骤：

1. `h1.innerHTML = "<h1>123</h1>"`设置元素内容
2. 将`<h1>123</h1>`解析为`HTML`文档
3. 用`DocumentFragment`将这个文档结构变成`DOM`节点
4. 用原本父元素上所有的内容然后替换成这个`DOM`节点

## 🔢 innerHTML 安全问题
`HTML5`和现代浏览器都会阻止通过`innerHTML`嵌入`script`脚步的程序执行。

```js
// 执行后页面会空白
document.documentElement.innerHTML= "<script>alert(123)</script>";
```

## 🔢 textContent/innerText
`textContent`和`innerText`都表示设置或获取元素文本内容。

当我们给元素插入的内容是纯文本的时候，要避免使用`innerHTML`！！！

`textContent`和`innerText`的区别在于获取到的内容不同

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
    <div class="box">
      <style>
        .content {
          color: orange;
        }
      </style>
      <p>这是p文本</p>
      <br />
      <p>这是p文本</p>
      <br />
      <span style="display: none">这是隐藏内容</span>
    </div>
  </body>
</html>
```

```js
var oBox = document.getElementsByClassName("box")[0];

console.log(oBox.textContent);
console.log(oBox.innerText);
```

![oBox.textContent](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1655273540972-196ae9bf-3ba8-48f2-b4e1-30a25c23a019.png)![oBox.innerText](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1655273552263-49e83dc8-2cc3-401b-b6fe-b753dbb35096.png)

`textContent`返回所有的内容包括`style`、`script`、设置隐藏的内容，而`innerText`只会返回类似浏览器页面上真正显示的内容。
