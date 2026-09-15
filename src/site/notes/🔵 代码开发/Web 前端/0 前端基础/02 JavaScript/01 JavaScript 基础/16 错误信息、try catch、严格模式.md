---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/16 错误信息、try catch、严格模式/","dg-note-properties":{}}
---

---
---
## 🔢 错误信息
## 🔢 1、SyntaxError 语法错误
例如变量名以数字开头

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420614399-a0bd96da-0c90-4297-9c1c-1ec4ca683fe0.png)

关键字赋值

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420668227-88815c7a-c5a7-49f3-8a03-4865b0c95cf6.png)

基本语法错误

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420703311-bf65eb02-7c4e-4903-9eda-815da12261e6.png)

## 🔢 2、引用错误
变量或者函数未被声明然后直接调用

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420803362-68b3b286-fb8e-4609-b591-a6243dffd391.png)

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420789288-dfae3d91-3eb8-4594-a6a1-dcd57b7a5b36.png)

给无法赋值的数据进行赋值

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420841466-f59c1356-4c22-42d5-a9f6-713c488e02be.png)

## 🔢 3、RangeError 范围错误
数组长度为负数的时候

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420900529-cb618d9d-2409-4f8c-bcef-cdaa79c5f6c7.png)

## 🔢 4、TypeError 类型错误
调用不存在的方法

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652420973521-5d0fb37f-71de-417f-b833-6a8413b0959e.png)

实例化原始值

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652421003691-49802552-46d8-407a-897e-b6e498158a18.png)

## 🔢 5、URIError
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1652421139481-65453c7b-f6d6-40e1-a191-9581dcb8fded.png)

## 🔢 6、eval() 函数执行错误

## 🔢 自定义错误类型
```js
new Error("代码错误");

// Error: 代码错误
//    at <anonymous>:1:1
```

## 🔢 try...catch...finally...
当`try`代码块内发生报错，程序不会停止执行会直接到`catch`中继续执行。

`catch`代码块的主要作用就是捕获`try`块中的错误然后执行相关的逻辑。

`finally`不管`try`有没有报错，也不管`catch`中有什么处理逻辑，`finally`内都会执行。

```js
try {
  console.log("正常执行1");
  console.log(a);
  console.log("正常执行2");
} catch (error) {
  console.log(error); // 字符串的报错提示
  console.log(error.name); // 错误的类型
  console.log(error.message); // 错误的具体的错误内容
  console.log("正常执行3");
} finally {
  // 不管try有没有错误，不管catch里面有什么操作，finally都会执行
  console.log("正常执行4");
}
console.log("正常执行5");

// 正常执行1
// VM9547:7 ReferenceError: a is not defined
//    at <anonymous>:3:17
// ReferenceError
// a is not defined
// 正常执行3
// 正常执行4
// 正常执行5
```

还可以配合`throw`来抛出异常信息。

```js
var jsonStr = "";
try {
  if (jsonStr === "") {
    // 抛出错误信息
    throw "JSON字符串为空";
  }
  console.log("我要执行了！");
  var json = JSON.parse(jsonStr);
  console.log(json);
} catch (error) {
  console.log(error);
  var errorTip = {
    name: "数据传输失败！",
    errorCode: "10010",
  };
  console.log(errorTip);
}

// JSON字符串为空
// {name: '数据传输失败！', errorCode: '10010'}
```

当`throw`直接抛出一个字符串错误的时候，是拿不到`name`和`message`属性的。

```js
var jsonStr = "";
try {
  if (jsonStr === "") {
    throw "JSON字符串为空";
  }
} catch (error) {
  console.log(error);
  console.log(error.name);
  console.log(error.message);
}

// JSON字符串为空
// undefind
// undefind
```

```js
var jsonStr = "";
try {
  if (jsonStr === "") {
    throw new Error("JSON字符串为空");
  }
} catch (error) {
  console.log(error);
  console.log(error.name);
  console.log(error.message);
}

// Error: JSON字符串为空
//    at <anonymous>:4:11
// Error
// JSON字符串为空
```

## 🔢 严格模式
`JavaScript`是由`ECMAScript`、`DOM`和`BOM`三部分组成的，「严格模式」指的是`ECMAScript5.0`版本后的语法、方法规范。

`ECMAScript`的历史

- 1997年，1.0 版本发布
- 1998年，2.0 版本发布
- 1999年，3.0版本发布
- 2007年，4.0提出草案，但是因为技术太过激进，多数浏览器厂商不同意这个草案
- 2008年，4.0版本中止，部分规范搬到 3.1 版本中，后 3.1 版本直接更名为 5.0 版本发布
- 2009年，5.0版本发布，5.0 版本没有对 3.0 版本进行太大的改动
- 2011年，5.1版本发布
- 2013年，6.0提出草案
- 2015年，6.0版本发布

<br/>

当时的`ECMAScript3.0`版本的语言安全性比较低，且效率一般，另外加上`ECMAScript5.0`大火，所以新增「严格模式」。

某些 3.0 语法（包括非 3.0 语法）在正常模式下运行正常，但是在`ES5`的严格模式下运行就会报错或者运行不正常。

启动「严格模式」的方式是使用`"use strict"`字符串。

```js
// 启用全局严格模式
"use strict";
function test() {
  // 函数内启用严格模式
  "use strict";
}
```

举例子🌰 ：

正常模式下，`a`直接赋值属于`window`对象的属性，严格模式下`a`会提示未定义。

```js
"use strict";
// Uncaught ReferenceError: a is not defined
a = 1;
```

正常模式下，函数内部的`this`指向`window`对象，严格模式下`this`对象是`undefind`

```js
"use strict";
function test(){
  console.log(this); // undefined，非严格模式下 this 指向 window
}
test();
```

正常模式下，`console.log(a)`正常输出`1`，严格模式下提示`a`是未定义。

```js
"use strict";
eval("var a = 1; console.log(a);");
// a is not defined，严格模式下 eval 是有独立的作用域
console.log(a);
```
