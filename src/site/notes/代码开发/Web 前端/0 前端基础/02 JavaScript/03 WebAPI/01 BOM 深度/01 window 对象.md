---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/01-bom/01-window/","dg-note-properties":{}}
---

`BOM`的核心是`window`对象，表示浏览器的实例。`window`对象在浏览器中有两重身份，一个是`ECMAScript`中的`Global`对象，另一个就是浏览器窗口的`JavaScript`接口。这意味着网页中定义的所有对象、变量和函数都以`window`作为其`Global`对象，都可以访问其上定义的`parseInt()`等全局方法。

```js
var age = 29;
var sayAge = () => alert(this.age);

alert(window.age); // 29
sayAge(); // 29
window.sayAge();  // 29
```

如果使用`let`或`const`替代`var`，则不会把变量添加给全局对象。

另外，访问未声明的变量会抛出错误，但是可以在`window`对象上查询是否存在可能未声明的变量。

```js
// 这会导致抛出错误，因为 oldValue 没有声明
var newValue = oldValue;

// 这不会抛出错误，因为这里是属性查询
// newValue 会被设置为 undefined
var newValue = window.oldValue;
```

`JavaScript`中有很多对象都暴露在全局作用域中，比如`location`和`navigator`，因而它们也是`window`对象的属性。

## 🔢 浏览器窗口打开/关闭

`window.open()`用来打开一个指定`url`的窗口。

参数1：要打开的`url`

参数2：窗口的`name`

参数3：窗口属性（[详见点击](https://www.runoob.com/jsref/met-win-open.html)）

参数4：新窗口在浏览器历史记录中是否替代当前加载页面的布尔值

📌 `window.open()`方法返回一个对新建窗口的引用。这个对象与普通`window`对象没有区别，只是为控制新窗口提供了方便。

<br/>

```js
widnow.open('url', 'page2', 'width=500,height=500', false);

// 窗口的 name 也可以是个特殊的 name
// 可选值有 _self、_parent、_top、_blank
window.open('url', '_blank', 'width=500,height=500', false);

var win = widnow.open("https://baidu.com", "width=500,height=500");
console.log(win);
```

`window.close()`用来关闭`window.open()`打开的窗口。

<br/>

```js
window.close();
```

`window`对象还有`name`、`opener`和`closed`属性可以获取`window`窗口的信息。

```js
// 获取 window.name
var win = window.open('url', 'test', 'width=500,height=500', false);
console.log(win.name)

// window.opener 指向打开新窗口的窗口
window.name = "page1";
var win = window.open("","page2");
console.log(win.opener.name, win.name); // page1 page2

// closed 表示新窗口是否关闭
var win = window.open("","page2");
if(win.closed){
  // do...
}
```

## 🔢 窗口的位置
<br/>color1
`window.moveTo(坐标x, 坐标y)`

`window.moveBy(向下移动的像素, 向右移动的像素)`

<br/>

```js
var win = widnow.open("https://baidu.com", "width=500,height=500");
win.moveTo(200, 200);
win.moveBy(10, 10);
```

## 🔢 窗口的关系
<br/>color5
`window.top`获取最上层窗口

`window.parent`获取当前窗口的父窗口

`window.self`获取当前窗口（之所以还要暴露 self，就是为了和 top、parent 保持一致）

<br/>

举个例子 🌰：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/01%20BOM%20%E6%B7%B1%E5%BA%A6/_assets/1656049360326-50ab8df1-c3f1-4b10-9047-e79bf880ca97.png)

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
    <h2>这是Page1</h2>
    <iframe src="./page2.html"></iframe>
  </body>
</html>
<script>
    window.name = "page1";

    console.log(window.top.name); // page1
    console.log(window.parent.name); // page1
    console.log(window.self.name); // page1
  </script>

```

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
    <h2>这是Page2</h2>
    <iframe src="./page3.html"></iframe>
  </body>
</html>
<script>
  window.name = "page2";

  console.log(window.top.name); // page1
  console.log(window.parent.name); // page1
  console.log(window.self.name); // page2
</script>

```

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
    <h2>这是Page3</h2>
  </body>
</html>
<script>
  window.name = "page3";

  console.log(window.top.name); // page1
  console.log(window.parent.name); // page2
  console.log(window.self.name); // page3
</script>

```

## 🔢 窗口的尺寸
## 🔢 获取
<br/>color2
`window.innerHeight/window.innerWidth`返回浏览器窗口可视窗口的高/宽度（不包含浏览器边框和工具栏）

`window.outerHeight/window.outerWidth`返回浏览器窗口可视窗口的高/宽度（包含浏览器边框和工具栏）

<br/>

<br/>warning
📌 `IE8`及以下版本浏览器不支持！

<br/>

```js
console.log(window.innerHeight);
console.log(window.innerWidth);

window.log(window.outerHeight);
window.log(window.outerWidth);
```

## 🔢 操作大小
<br/>color2
`resizeTo(宽度值, 高度值) `

`resizeBy(宽度像素, 高度像素)`

<br/>

```js
var win = widnow.open("https://baidu.com", "width=500,height=500");
win.resizeTo(200, 200);
win.resizeBy(10, 10);
```

## 🔢 窗口的滚动距离
## 🔢 获取
<br/>success
`window.pageXoffset`返回浏览器`X`轴滚动的距离

`window.pageYoffset`返回浏览器`Y`轴滚动的距离

<br/>

<br/>warning
📌 `IE8`及以下版本浏览器不支持！

<br/>

```js
console.log(window.pageXoffset);
console.log(window.pageYoffset);
```

## 🔢 操作距离
<br/>success
`window.scrollTo(滚动X坐标, 滚动Y坐标)`，滚动到指定位置（绝对位置）。

`window.scrollBy(滚动X像素, 滚动Y像素)`，相对于当前滚动位置移动（相对位置）。

`window.scroll(滚动X坐标, 滚动Y坐标)`，本质上是`scrollTo`的别名。

<br/>

`X`、`Y`这两个参数在前两个方法中表示要滚动到的坐标，在最后一个方法中表示滚动的距离。

`window.scrollBy(x, y);`该方法的距离是累加的，每次滚动会在已滚动的距离上加上需要设置的滚动距离

```js
// 滚动到页面左上角
window.scrollTo(0, 0);

// 当前视口向下滚动 100 像素
window.scrollBy(0, 100);
```

这几个方法也都接收一个`ScrollToOptions`字典，除了提供偏移值，还可以通过`behavior`属性告诉浏览器是否平滑滚动。

```js
// 正常滚动
window.scrollTo({
  left: 100,
  top: 100,
  behavior: 'auto'
});

// 平滑滚动
window.scrollBy({
  left: 100,
  top: 100,
  behavior: 'smooth'
});
```

<br/>warning
⚠️ 注意

`scroll()`、`scrollTo()`、`scrollBy()`这三个属性在现代浏览器中也存在于 Element 属性中。

<br/>

```js
document.getElementById("target").scrollTo(0, 500);
document.getElementById("target").scrollBy(0, 500);
document.getElementById("target").scroll(0, 500);
```

## 🔢 系统对话框
<br/>warning
`window.alert("提示的内容")`

该方法在使用的时候可以不带`window`

页面上显示一个提示框，会阻塞程序的执行

<br/>

```js
window.alert("输入错误！");
alert("输入错误！");
```

<br/>warning
`window.confirm("提示的内容")`

该方法在使用的时候可以不带`window`

页面上显示一个确认框，返回一个布尔值，会阻塞程序的执行

<br/>

```js
let res = window.confirm("确认删除？");
let res = confirm("确认删除？");
if(res){
  // del...
}
```

<br/>warning
`window.prompt("提示的内容")`

该方法在使用的时候可以不带`window`

页面上显示一个输入框，返回输入的字符串，会阻塞程序的执行

<br/>

```js
let res = window.prompt("请输入姓名");
let res = prompt("请输入姓名");

console.log(res);
```

<br/>warning
`window.print()`

该方法在使用的时候可以不带`window`

将页面进行打印

<br/>

```js
window.print();
print()
```

## 🔢 定时器
`JavaScript`在浏览器中是单线程执行的，但允许使用定时器指定在某个时间之后或每隔一段时间就执行相应的代码。

## 🔢 setTimeout()
<br/>color3
表示一定时间后做某事（延迟器）

返回一个延时的`ID`

参数1：要执行的回调函数

参数2：要延迟的时间（毫秒）

<br/>

```js
// 在 1 秒后显示警告框
setTimeout(() => alert("Hello world!"), 1000);

setTimeout(function(){
 // do...
}, 2000);
```

> JavaScript 是单线程的，所以每次只能执行一段代码。为了调度不同代码的执行，JavaScript 维护了一个任务队列。其中的任务会按照添加到队列的先后顺序执行。setTimeout() 的第二个参数只是告诉 JavaScript 引擎在指定的毫秒数过后把任务添加到这个队列。如果队列是空的，则会立即执行该代码。如果队列不是空的，则代码必须等待前面的任务执行完才能执行。
>

## 🔢 clearTimeout()
<br/>color3
用于取消销毁`setTimeout()`延时器

参数1：`setTimeout()`返回的`ID`

<br/>

```js
// 设置超时任务
let timeoutId = setTimeout(() => alert("Hello world!"), 1000);

// 取消延时器
clearTimeout(timeoutId);
```

> 注意 ⚠️
>
> 所有超时执行的代码（函数）都会在全局作用域中的一个匿名函数中运行，因此函数中的 `this`值在非严格模式下始终指向 window，而在严格模式下是 undefined。如果给`setTimeout()`提供了一个箭头函数，那么`this`会保留为定义它时所在的词汇作用域。
>

## 🔢 setInterval()
<br/>color3
表示每隔一定时间后做某事，直到定时器被取消销毁（定时器）

返回一个定时器的`ID`

参数1：要执行的回调函数

参数2：要定时等待的时间（毫秒）

<br/>

```js
setInterval(() => alert("Hello world!"), 10000);
```

> 注意 ⚠️
>
> 这里的关键点是，第二个参数，也就是间隔时间，指的是向队列添加新任务之前等待的时间。比如，调用 setInterval() 的时间为 01:00:00，间隔时间为 3000 毫秒。这意味着 01:00:03 时，浏览器会把任务添加到执行队列。浏览器不关心这个任务什么时候执行或者执行要花多长时间。因此，到了 01:00:06，它会再向队列中添加一个任务。由此可看出，执行时间短、非阻塞的回调函数比较适合 setInterval()。
>

## 🔢 clearInterval()
<br/>color3
用于取消销毁`setInterval()`定时器

参数1：`setInterval()`返回的`ID`

<br/>

```js
// 设置延迟执行任务
let timeoutId = setInterval(() => alert("Hello world!"), 1000);

// 取消超时任务
clearInterval(timeoutId);
```

<u></u>

<u>相对于</u>`<u>setTimeout()</u>`<u>而言，取消定时的能力对</u>`<u>setInterval( )</u>`<u>更加重要。毕竟，如果一直不管它，那么定时任务会一直执行到页面卸载。</u>
