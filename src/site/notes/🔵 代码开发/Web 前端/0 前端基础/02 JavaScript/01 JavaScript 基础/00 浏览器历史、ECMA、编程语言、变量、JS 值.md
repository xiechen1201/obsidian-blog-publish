---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/00 浏览器历史、ECMA、编程语言、变量、JS 值/","dg-note-properties":{}}
---

---
---
## 🔢 1、浏览器的历史和JS的诞生
![画板](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1675923741413-9ce07d83-c8a7-4bb5-994f-3b81f456705c.jpeg)

## 🔢 🔢
## 🔢 2、五大浏览器
| 浏览器 | 内核 |
| --- | --- |
| IE | trident |
| chrome | webkit、blink |
| safari | webkit |
| firefox | gecko |
| opera | presto |

<br/>color5
浏览器的内核可以分为「渲染引擎」和「`JS`解析引擎」等！！！

<br/>

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1666698859590-e56ba7b5-2db0-4e40-b849-237757f24da9.png)

## 🔢 3、ECMA
`ECMA`的全称是 `European computer manufacturers association` 欧洲计算机制造联合会，该联合会主要是：评估、开发、认可电信、计算机的标准。

它管理了许多语言的规范，其中`ECMA-262` 是脚本语言的规范。

## 🔢 4、编程语言
编程语言分为「编译型」和「解释型」语言。

他们执行的过程如下：

编译型：源码 --》编译器 --》机器语言 --》可执行的文件

解释型：源码 --》解释器 --》解释一行执行一行

`JavaScript`是客户端脚本语言，类似的`PHP`是服务端脚本语言；

`JavaScript`包含`ECMAScript`、`DOM`、`BOM`三大部分；

`JavaScript`引擎是单线程执行；

## 🔢 5、JavaScript 的使用
1、写在 `script`标签中

```html
<script>
  function sayHi() {
    console.log("Hi!");
  }
</script>
```

2、使用外部的`JavaScript`文件

```html
<script src="example.js"></script>
```

值得注意的是在引入外部JS文件的时候，`script`标签内不可写其他的代码。

```html
<script src="example.js">
  // 禁止
  console.log("Hello World");
</script>
```

## 🔢 6、变量
要定义变量可以使`var`关键字：

```js
var massage;  // 声明变量
massage = "test";  // 变量赋值
var a = 123;  // 声明变量且赋值
b = 456;  // 合法，但是不推荐！！！
```

连续声明多个变量：

```js
var x = 1,
    y = 2,
    z;
```

命名规范：

声明变量时的名字是有规范要求的

1、不能以数字开头

2、可以用字母、_、$ 开头

3、可以包含字母、_、$

4、不能使用关键字和保留字 [点击查看](https://www.yuque.com/xiechen/ga1rqp/gd84e1)

5、语义化、避免中文的拼音

4、小驼峰和大驼峰

## 🔢 7、JS 的值
JS 的值分为「原始数据」和「引用数据」

原始数据包括：`number`、`string`、`boolean`、`undefined`、`null`、`symbol`

引用数据包括：`object`

> `object`又包含：`Object`、`Array`、`Function`、`Date`、`Regexp`等
>

为什么`JavaScript`不像`Java`那样声明的时候要规定数据类型？

```java
int a = 1;
```

```java
var a = 1;
```

因为`JavaScript`是弱类型语言，`a`的数据类型是在`=`后面进行判断的，也就是`1`是`number`类型。

## 🔢 8、栈和堆
![画板](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1646726077435-f5712886-49f5-457d-9a27-47f262b1f695.jpeg)

总结：

原始数据的值都是保存在栈内存中，且相互不会影响。

引用数据的值都是保存在堆内存中，栈内存中只保存了堆内存的地址，当两个引用类型保存同一个堆内存地址时，更改数据会相互影响。

<br/>
