---
{"dg-publish":true,"permalink":"//web/1/02-webpack4/02-css/00-css/","dg-note-properties":{}}
---

## 🔢 CSS 存在的问题
当我们的项目开发到一定规模的时候，我们不仅仅希望只把 JS 代码进行细分，同样希望能把 CSS 代码也进行细分，然后 CSS 没有太多的演化升级过程来解决模块细分的问题，这就需要使用一些构建工具来进行处理。

## 🔢 类名冲突
当我们写一个 CSS 类的时候，我们可能会写一个全局的类或者书写多个层级的类。

但是，这两种方法都存在一定的问题：

1、过深的层级不利于编写、阅读、压缩和复用

```css
.a .b .c .d .e .f .g .box{
  background-color: red;
}
```

2、过浅（或全局）的层级又容易导致类名冲突

```css
.a .box{
  background-color: green;
}

.box{
  background-color: blank;
}
```

一旦样式多起来的时候，这个问题就会变的非常的严重，归根结底就是因为类名冲突不好解决的问题。

## 🔢 重复样式
如果我们的代码中有一些样式都是重复的，每次出现都需要重写一次，例如一个全局的颜色。如果某天这个颜色需要更改为其他的颜色值，那面就需要全局搜索然后一个个的修改，及其的不好维护。

## 🔢 CSS 文件细分
在大型的项目开发中，CSS 文件也需要更加精细的拆分，这样有利于 CSS 代码的维护。

例如，做一个轮播图的模块，它不仅需要依赖 JS 文件，还需要依赖 CSS 的样式，既然依赖的 JS 功能仅关心轮播图，那 CSS 样式应该也只关心轮播图即可。

同样的，不同的功能需要依赖的不同的 JS 文件，公共样式可以进行单独的抽离，这样就需要把 CSS 文件进行更加精细的拆分。

CSS 文件拆分应该和 JS 文件一样，当我们运行在真实的生产环境下时 CSS 文件应越少越好，这样可以减少一定的网络请求。

综上，CSS 和 JS 一样同样需要进行工程化的管理。

## 🔢 如何解决？
这么多年，官方一直没有提出方案来解决上面三个问题，于是一些第三方机构针对不同的问题，提出了自己的解决方案。

## 🔢 解决类名冲突问题
1、命名约定

通过一些命名的标准，来解决类名冲突的问题，常见的标准有：

- BEM
- OOCSS
- AMCSS
- SMACSS
- ...

就拿 BEM 来举例，一个完整的 BEM 类名应该是`block__element_modifier`，例如`banner__dot_selected`。

三个部分具体的含义：

- block：表示页面中的大区域表示嘴顶级的划分。例如轮播图`banner`、布局`layout`、文章`article`等
- element：表示区域中的组成部分，例如轮播图的图片`banner_img`、轮播图的容器`banner_container`等
- modifier：表示状态，例如展开的侧边栏`layout_left_expand`、处于选中状态的圆点`banner_dot_selected`

在某些大工程中，如果使用 BEM 规范还可能需要加上一些前缀来表示用途：

- l：layout，表示是用来布局的
- c：component，表示这是一个组件
- u：util，表示这个样式是通用的，工具性质的
- j：javascript，表示这个类要被 JS 所获取

但是，只要是人写的代码就一定会存在问题，可能有的人偷懒就不去遵守规范又会出现问题。

---

2、css in js

从名字就能看出来意为把 CSS 放在 JS 代码中。改方案认为 CSS 已经无可救药了，干脆直接使用 JS 对象来表示样式，然后把样式直接应用到内联 style 中去。

这样一来就可以：

- 通过一个函数返回一个样式对象
- 把公共的样式提取到公共模块中返回
- 应用 KS 的各种特性来操作对象，例如混合、提取、拆分
- ...

```js
import { applyStyles } from "../utils/index.js";

const div1 = document.getElementById("div1");
const div2 = document.getElementById("div2");

const styles = {
  backgroundColor: "#f40",
  color: "#fff",
  width: "400px",
  height: "500px",
  margin: "0 auto",
  border: "2px solid #333"
}

applyStyles(div1, styles);
applyStyles(div2, styles);
```

由于这种描述样式的方式根本就不存在类名，自然不会有类名冲突。

不过，这个方案太过激进。

---

3、css module

目前为止，我们发现通过命名规范来限制类名太过死板，而 css in js 虽然足够灵活，但是书写不便。 css module 开辟一种全新的思路来解决类名冲突的问题。

这是一种非常有趣和好用的 CSS 模块方案，编写简单，绝对不重名。

css module 的思路：

- css 的类名冲突一般都发生在大型项目中
- 大型项目往往都会使用构建工具（例如 Webpack）
- 构建工具容许把 CSS 分为更加精细的模块
- 和 JS 变量一样，每个 CSS 模块中都难以出现冲突的类名，类名冲突一般发生在多个模块合并在一起的时候
- 只需要保证构建工具在合并样式后不会产生类名冲突即可

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695613533234-31c755a1-cfaf-4b53-ae1e-0179ea68d79a.png)

具体操作请看下一章中的《Webpack 打包 CSS 文件》

## 🔢 解决重复样式问题
1、css in js

和上面说的是一个方案，这个方案太激进，很多习惯书写 CSS 的开发者非常的不适应。

2、预编译器

有一些第三方机构搞出一套新的 CSS 语言来解决这个问题。它支持变量、函数、混入等高级语法，然后通过相关的编译工具把代码转换为正常的 CSS 代码。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/02%20CSS%20%E5%B7%A5%E7%A8%8B%E5%8C%96/_assets/1695630348828-bd55a6d3-0de3-4e61-88f8-cb72495adcd2.png)

例如 Less、Sass 都是预编译器。

less官网：[http://lesscss.org](http://lesscss.org)

less中文文档1（非官方）：[http://lesscss.cn](http://lesscss.cn/)

sass官网：[https://sass-lang.com](https://sass-lang.com)

sass中文文档1（非官方）：[https://www.sass.hk](https://www.sass.hk/)

具体操作请看下一章中的《Webpack 打包 CSS 文件》

## 🔢 解决 CSS 文件细分问题
使用构建工具，例如 Webpack 结合一些 Loader 和 Plugin 来完成打包、合并、压缩等工作。

具体操作请看下一章中的《Webpack 打包 CSS 文件》
