---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/01 CSS3/10 Flex 弹性盒模型/","dg-note-properties":{}}
---

---
---
弹性盒模型（Flexible Box），能够让子元素之间提高空间分布和对齐能力。

如果想要将一个普通的盒子设置为弹性盒，需要使用`display`属性：

```css
.box{
  /*  转换为弹性盒模型  */
  display: flex;

	/*  兼容写法  */
  display: -webkit-box;
  display: -webkit-flex;
  display: -ms-flexbox;
}
```

<br/>warning
⚠️ 注意

设为 Flex 布局以后，Flex Item（弹性元素）的`float`、`clear`、`vertical-align`属性将失效。

<br/>

在正式学习弹性盒模型之前我们需要了解几个概念：

1、弹性布局是一种一维布局，一次只能处理一条线的布局，我们称为「主轴」，和主轴相交叉的轴我们称为「交叉轴」（或侧轴）。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667134216322-be75615b-9e14-49a7-a9cb-8dfd2f03f9fa.png)

2、弹性容器 Flex Contanier 下所有的子成员都会变为 Flex Item（弹性项目）。

## 🔢 弹性容器属性
## 🔢 flex-direction
用于设置弹性容器的主轴方向，默认是横向的！

语法：

```latex
flex-direction: value;
```

属性值：

- value 可选：
    - `row`水平，默认值；
    - `row-reverse`水平反转；
    - `column`垂直；
    - `column-reverse`垂直反转；

示例：

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
}
```

![row](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667134478388-0c4a87cf-eb71-40a6-bb2d-e2539419d785.png)

![row-reverse](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667134498706-23818ed7-c4d3-436b-abaf-744d4d44cf30.png)

![column](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667134519971-274093c2-ab10-4250-aea5-99a780e1cde8.png)

![column-reverse](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667134541072-6edf4793-85e2-436c-9aa1-6df3f195bea8.png)

## 🔢 flex-wrap
当弹性容器内有多个子项目占不下一行的时候，会把所有的子项目进行压缩，强制在一行，该属性就是用于设置是否换行。

语法：

```latex
flex-wrap: value;
```

属性值：

- value 可选：
    - `nowrap`子项目不换行，压缩弹性项目的大小，默认值；
    - `wrap`子项目换行；
    - `wrap-reverse`换行反转；

示例：

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap-reverse;
}
```

![nowrap](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667134989228-467cf617-62ac-4218-9644-0511cdadc605.png)

![wrap](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667135055591-ff1048f6-7bf0-4ce6-a120-2e780c255330.png)

![wrap-reverse](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667135077207-c75236c2-59f9-44b6-a909-18d555c7df3f.png)

## 🔢 flex-flow
该属性是`flex-direction`和`flex-wrap`两个属性的复合属性。

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-flow: row wrap-reverse;
}
```

## 🔢 justify-content
该属性用于设置子项目在弹性容器内在主轴上的对齐方式。

语法：

```latex
justify-content: value;
```

属性值：

- value 可选：
    - `flex-start`沿着主轴起点对齐，默认值；
    - `flex-end`沿着主轴终点对齐；
    - `center`沿着主轴居中对齐；
    - `space-between`沿着主轴两端对齐；
    - `space-around`沿着主轴弹性项目两侧间距相等，平分弹性容器空间；
- 示例：

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-around;
}
```

![flex-start](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667135369757-dd9db622-24ea-466a-b7ce-5ec6380eb128.png)

![flex-end](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667135390827-a094734b-4118-45e6-bf26-00b270a8bf2c.png)

![center](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667135410286-fb86a1eb-a89e-4fe0-b7b9-a00eef24f20a.png)

![space-between](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667135430949-ef1015e0-e6ed-4db2-8f71-0bcaa0c5fbb4.png)

![space-around](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667135450985-c63fb652-f22f-4f96-9645-97cc4312449d.png)

## 🔢 align-items
该属性用于设置「单列」弹性项目在交叉轴（侧轴）上的对齐方式。

语法：

```latex
align-items: value;
```

属性值：

- value 可选：
    - `flex-start`沿着侧轴起点对齐；
    - `flex-end`沿着侧轴终点对齐；
    - `center`居中对齐；
    - `baseline`基准线；
    - `stretch`没有高度时拉伸铺满；

示例：

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: flex-start;
}
```

![flex-start](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136243370-e9f20877-0607-4865-a210-4eb0c1a0bb0a.png)

![flex-end](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136283240-b7bc6c97-2820-4059-b7d8-b60810a45afc.png)

![center](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136304415-4195dff1-6115-4979-b202-1609f131386e.png)

![baseline](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136453952-06497aeb-75ef-4e2d-bcfc-08a01c107b19.png)

![stretch](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136778118-b575ad23-55f9-4213-9fae-072fa081f4c8.png)

## 🔢 align-content
定义多根主轴线的对齐方式，如果父元只有一根轴线，该属性不起作用，必须设置`flex-wrap: wrap;`。

语法：

```latex
align-content: value;
```

属性值：

- value 可选：
    - `flex-start`沿着侧轴起点对齐；
    - `flex-end`沿着侧轴终点对齐；
    - `center`沿着侧轴居中对齐；
    - `space-between`沿着侧轴两端对齐；
    - `space-around`沿着侧轴弹性项目上下间距相等，平分弹性容器空间；
    - `stretch`默认值，多轴线平分占满整个交叉轴；

示例：

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-content: flex-start;
}
```

![flex-start](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136831602-bc7a11c4-36fd-4fa2-ab58-e39618a65126.png)

![flex-end](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136859803-8a1f492f-a467-485c-8c89-11c42eed8d0a.png)

![center](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136888438-3dcbc8c1-36e0-4711-be59-2f825e3f891f.png)

![space-between](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136913047-9842ab6a-5157-490a-a932-9a7b3d0bfbb3.png)

![space-around](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136935237-5e3d4a24-cda9-4344-a91f-f7a94b3a2842.png)

![stretch](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667136958807-6110d6dd-5026-40ba-b757-2494b7dac90d.png)

## 🔢 弹性项目属性
## 🔢 flex-grow
该属性用于设置弹性项目放大比例，前提弹性容器有空间剩余才会生效！

语法：

```latex
flex-grow: number;
```

属性值：

- number：一个数字，0 为默认值表示不放大，1 为占据整个剩余空间，如果多个弹性项目设置该属性，那么会均分占据剩余空间；

示例：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667137527343-e77e1a02-d501-4832-8bfd-b1036424283a.png)

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
}
.right {
  background-color: green;
  flex-grow: 1;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667137589513-e764b286-d1f9-4045-bf02-f854c1b37e6e.png)

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
}
.mid {
  background-color: orange;
  flex-grow: 1;
}
.right {
  background-color: green;
  flex-grow: 1;
}
```

示例，如果有一个宽度为`1000px`的父元素下有 3 个子元素，宽分别是`100px`、`200px`、`300px`，然后分别设置了`flex-grow: 1/2/3`，那么每个子元素最终分配多少空间呢？

首先，要先计算一下 3 个子元素已知的占用空间：100 + 200 + 300 = 600，这样就知道还剩下`400px`的空间可以让子元素进行伸缩。

然后就需要根据每个子元素的`flex-grow`值来进行计算，分别是：

- 400 × 1 / 6 = 66.67px
- 400 × 2 / 6 = 133.33px
- 400 × 3 / 6 = 200px

加上初始的宽度，3 个子元素的宽度最终是：

- 100 + 66.67 = 166.67px
- 200 + 133.33 = 333.33px
- 300 + 200 = 500px

## 🔢 flex-shrink
该属性用于设置弹性项目缩小比例，前提是空间不足的时候才会缩小。

语法：

```latex
flex-shrink: number;
```

属性值：

- number：一个数字，0 表示回归默认值大小，1 为默认值，值越大缩放的越小；

示例：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667137794438-a43e2b9a-060c-465f-bf49-849a4543a5fc.png)

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
}
.left {
  background-color: blue;
  flex-shrink: 0;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667137846008-aecdc3f0-6808-430c-93be-4dd9f257888a.png)

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
}
.left {
  background-color: blue;
  flex-shrink: 2;
}
```

## 🔢 flex-basis
放大和缩小都是按照基准线来计算的，该属性就是更改基准值的。

语法：

```latex
flex-basis: value;
```

属性值：

- value 可选：
    - `auto`也就是元素本身的宽度；
    - 具体宽度像素；

## 🔢 flex
该属性是`flex-grow`、`flex-shrink`和`flex-basis`的复合值。

```css
.left {
  background-color: blue;
  flex: 0 1 auto; /* 1 和 auto 是可选的 */
}
```

## 🔢 order
设置弹性项排序优先级。

语法：

```latex
order: number;
```

属性值：

- number：数字越大越往后排，默认为 0，支持负数；

示例：

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
}
.box > div {
  width: 100px;
  height: 100px;
  color: #fff;
  font-size: 18px;
  text-align: center;
  line-height: 100px;
}
.left {
  background-color: blue;
  order: 2;
}
.mid {
  background-color: orange;
  order: 1;
}
.right {
  background-color: green;
  order: 3;
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667138156374-9a805839-4454-48f1-98f0-7d37cc47a40c.png)

## 🔢 align-self
单独设置单个弹性项目在侧轴的对齐方式。

语法：

```latex
align-self: value;
```

属性值：

- value 可选：
    - `flex-start`沿着侧轴起点对齐；
    - `flex-end`沿着侧轴终点对齐；
    - `center`沿着侧轴居中对齐；
    - `baseline`沿着侧轴基准线对齐；
    - `stretch`没有高度时拉伸铺满；
    - `auto`默认值，继承父容器的`align-items`属性；

示例：

```css
.box {
  width: 500px;
  height: 500px;
  border: 1px solid #000;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
}
.box > div {
  width: 100px;
  height: 100px;
  color: #fff;
  font-size: 18px;
  text-align: center;
  line-height: 100px;
}
.left {
  background-color: blue;
  line-height: 40px !important;
  align-self: baseline;
}
.mid {
  background-color: orange;
  line-height: 60px !important;
  align-self: baseline;
}
.right {
  background-color: green;
  line-height: 80px !important;
}
```

![flex-start](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667138313875-7afd718e-0973-43bb-8460-b9d3567b1433.png)

![flex-end](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667138331763-aef9df37-d510-490a-9f46-d3df18b93680.png)

![center](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667138352647-482e4484-1d71-4f9f-867b-792a9bf963b1.png)

![baseline](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667138569510-bc685fc7-2341-4b75-b96b-057090123323.png)

![stretch](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667138713838-93b28577-f425-480f-8252-3e1c6de8d1d4.png)

![auto](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/01%20CSS3/_assets/1667138736471-b15e9f40-3851-46a2-baa3-03ca83229fa3.png)
