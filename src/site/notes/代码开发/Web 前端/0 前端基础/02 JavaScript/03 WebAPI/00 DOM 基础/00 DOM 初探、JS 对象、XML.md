---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/00-dom/00-dom-js-xml/","dg-note-properties":{}}
---

`JavaScript`包括`ECMAScript`、`DOM`、`BOM`三大部分组成。

`DOM`的全称是`Document Object Model`文档对象模型。

`JavaScript`中有三种对象：

- 本地对象`Native Object`：和浏览器没太大关系的对象，例如：`Object`、`Function`、`Array`、`String`、`Number`、`Boolean`、`Error`、`Date`、`RegExp`...
- 内置对象`Bulit-in Object`：也称全局对象可以直接调用，例如：`isNaN()`、`parseInt()`、`Math.radom()`...
- 宿主对象`Host Object`：执行`JS`脚本的环境提供的对象，例如浏览器提供的对象就是宿主对象。宿主对象又称浏览器对象，不同的浏览器提供了不同的宿主对象的方法，所以不同的浏览器在使用宿主对象方法的时候会有兼容性。

浏览器对象有`Window`（`Window`所有的方法都是操作`BOM`的）和`Docment`（`Docment`所有的方法都是操作`DOM`的），`DOM`存在的目的就是通过浏览器提供的这一套方法表示或者操作`HTML`或者`XML`

什么是`XML`？

`XML`其实就是一种自定义的标签；而`HTML`是浏览器提供的；`XML`为`HTML`奠定了一个基本的`HTML`规范。

```xml
<body>
  <!-- XML -->
  <person>
    <name>张三</name>
    <sex>男</sex>
    <age>18</age>
  </person>
</body>
```

`DOM`无法更改元素的`CSS`样式表，操作的其实是元素的内联样式。

```xml
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
    <style>
      .test {
        width: 100px;
        height: 100px;
        background-color: red;
      }
    </style>
  </head>
  <body>
    <div class="test"></div>
  </body>
</html>
```

![更改前](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1652929923485-d6bc0d5a-22ee-4ce6-9a2c-702d9de33240.png)

![更改后](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/00%20DOM%20%E5%9F%BA%E7%A1%80/_assets/1652929990188-71308697-0584-4af9-a4e3-b11f79420dc4.png)
