---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/01 CSS3/09 BFC/","dg-note-properties":{}}
---

---
---
BFC 的全称是 块级格式化上下文（Block formating contexts)。

简单来说，它是页面中的一个独立的渲染区域，内部有一套自己的布局规则，决定了元素如何排列、对齐，以及它们与其他元素的关系。在进行可视化布局时，BFC 充当一个独立的环境，确保其中的 HTML 元素按照特定规则进行排列，而不会受到外部影响。

再简单一点：BFC 就是一个独立的布局环境，BFC 内部的元素布局于外部互不影响。

BFC 虽然是一个独立的布局环境，但是也不意味着布局没有章法，基本的规则还是要有的，布局规则如下：

- 内部的盒子会在垂直方向一个挨着一个的放置；
- 盒子垂直方向的距离由`margin`属性决定，属于同一个 BFC 的两个相邻的盒子的`margin`会发生重叠；
- 每个盒子的左外边框紧挨着包含块的左边框，即使浮动元素也是如此；
- BFC 的区域不会和浮动的盒子重叠；
- BFC 就是页面上一个隔离的独立容器，容器里的子元素不会影响到外面的元素，反之亦然；
- 计算 BFC 的高度时，浮动子元素也参与计算；

实际上在一个标准流中，`<body>`元素就是一个天然的 BFC。

如果是其他区域，要想单独设置成一个 BFC，那么可以通过下面的方式进行触发：

- `<html>`元素；
- 设置`float`属性；
- 设置`position`属性为`absolute/fixed`；
- 设置`overflow`属性为`auto/scroll/hidden`；
- 设置`dispaly`属性为`inline-block/table-cell`;
- 更多详见：[https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_display/Block_formatting_context](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_display/Block_formatting_context)

那么 BFC 到底有啥用处呢？我们看几个场景。

1、解决浮动元素使父元素高度塌陷的问题。

```html
<div class="father">
   <div class="son"></div>
</div>
```

```css
.father {
  border: 5px solid;
}

.son {
  width: 100px;
  height: 100px;
  background-color: blue;
  float: left;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1742795676200-fb07b6a7-b4e8-4c07-b9bc-20d085341b97.png)

上面的代码中，`.father`元素的高度是由`.son`元素撑起的，如果`.son`元素设置浮动后，那么父元素的高度就塌陷了。

此时，我们可以对父元素设置一个 BFC，例如：

```css
.father {
  border: 5px solid;
  /* 将父元素设置为一个 BFC */
  overflow: hidden;
}

.son {
  width: 100px;
  height: 100px;
  background-color: blue;
  float: left;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1742795738864-75124103-85ad-4dd8-aa43-946d1db84531.png)

由于父元素变成了 BFC，高度就没有产生塌陷了，其原因是在计算 BFC 高度的时候，浮动的子元素也参与计算。

2、非浮动元素被浮动元素覆盖。

```html
<div class="box1"></div>
<div class="box2"></div>
```

```css
.box1{
  width: 100px;
  height: 50px;
  background-color: red;
  float: left;
}

.box2{
  width: 50px;
  height: 50px;
  background-color: blue;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1742795870006-45bd537b-4053-40f3-8b9c-2bd510bdc8a7.png)

上面的代码中，由于`.box1`设置了浮动效果，所以就会脱离标准文档流，自然而然`.box2`就会往上面跑，结果就被`.box1`给覆盖了。

接下来我们给`.box2`设置 BFC：

```css
.box1{
  width: 100px;
  height: 50px;
  background-color: red;
  float: left;
}

.box2{
  width: 50px;
  height: 50px;
  background-color: blue;
  /* 设置一个 BFC */
  overflow: hidden;
}

```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1742796003074-ded9e79d-80c8-466d-aea2-947b468ca066.png)

可以看到，`.box2`的元素露出来了。

基于这个特点，我们可以制作一个两栏的自适应布局，方法就是给固定栏设置固定宽度，给不固定栏开启 BFC。

```html
<div class="left">导航栏</div>
<div class="right">这是右侧</div>
```

```css
*{
  margin: 0;
  padding: 0;
}

.left {
  width: 200px;
  height: 100vh;
  background-color: skyblue;
  float: left;
}

.right {
  width: calc(100% - 200px);
  height: 100vh;
  background-color: yellowgreen;
  /* 设置一个 BFC */
  overflow: hidden;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1742796147449-94be095c-4b45-401a-be31-e50cb05a7cb1.png)

就此实现了一个左侧宽度固定，右侧宽度自适应的两栏布局。

3、外边距垂直方向重合的问题。

```html
<div class="box1"></div>
<div class="box2"></div>
```

```css
* {
  margin: 0;
  padding: 0;
}

.box1{
  width: 100px;
  height: 100px;
  background-color: red;
  margin-bottom: 10px;
}

.box2{
  width: 100px;
  height: 100px;
  background-color: blue;
  margin-top: 10px;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1742796223808-54c13870-9c1d-48e3-854e-83f6dc527b65.png)

可以看到两个元素的外边距实际为 10px，此时可以给`.box2`外部在套一个`<div>`，并且将这个`<div>`设置为 BFC。

```html
<div class="box1"></div>
<div class="container">
  <div class="box2"></div>
</div>
```

```css
.container{
  overflow: hidden;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1742796329529-9205594c-e044-4b5f-9b04-a2e512843847.png)

这样可以让两个元素的外边距变成正常的 20px。

明白了 BFC 之后，那么其他的 IFC、GFC 和 FFC 也就大同小异了。

- IFC（Inline formatting context），行内格式化上下文，也就是一块区域以行内元素的形式来进行格式化；
- GFC（GrideLayout formatting contexts），网格布局格式化上下文，将一块区域以 Grid 网格的形式来进行格式化；
- FFC（Flex formatting contexts），弹性格式化上下文，将一块区域以弹性盒的形式来格式化；

> 更多关于格式化上下文的内容，可以参阅 MDN：
>
> + BFC：[https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context](https://gitee.com/link?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FGuide%2FCSS%2FBlock_formatting_context)；
> + IFC：[https://developer.mozilla.org/zh-CN/docs/Web/CSS/Inline_formatting_context](https://gitee.com/link?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2FInline_formatting_context)；
>
