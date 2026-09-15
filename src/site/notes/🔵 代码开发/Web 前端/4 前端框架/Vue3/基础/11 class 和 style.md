---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/Vue3/基础/11 class 和 style/","dg-note-properties":{}}
---

`class`和`style`都可以是模版的属性，它们都可以通过`v-bind`动态的绑定到模版中：

```html
<div :class="className">这是class</div>
<div :style="styleObj">这是style</div>
```

`Vue`对`v-bind`的`class`和`style`进行了特殊的封装，形式比较多，主要是对象和数组的绑定方式。

## 🔢 class

1、`class`属性绑定为一个对象

```js
const app = {
  template:`<div :class="{ active: isActive }"></div>`,
  // 渲染为 <div class="active"></div>
  data(){
    return {
      isActive: true
    }
  }
}
```

`active`类名是否生效，取决于数据属性`isActive` 的真假值。

或者直接绑定为一个对象：

```js
const app = {
  template:`<div :class="{ active: isActive, 'text-danger': true }"></div>`,
  // 渲染为 <div class="active text-danger"></div>
  data(){
    return {
      isActive: true
    }
  }
}
```

2、`class`属性绑定为一个数组

```js
const app = {
  template:`<div :class="[activeClass, errorClass]"></div>`,
  // 渲染为 <div class="active error"></div>
  data(){
    return {
      activeClass: 'active',
      errorClass: 'error'
    }
  }
}
```

或者在数组中使用三元运算符有条件的渲染某个值：

```js
const app = {
  template:`<div :class="[isActive ? activeClass : '', errorClass]"></div>`,
  // 渲染为 <div class="error"></div>
  data(){
    return {
      isActive: false,
      activeClass: 'active',
      errorClass: 'error'
    }
  }
}
```

3、`class`属性也可以直接给组件进行绑定

```js
const myComponent = {
  template: `<p class="foo bar">Hi!</p>`
  // 渲染为 <p class="baz boo foo bar">Hi!</p>
}
const app = {
  template: `<myComponent class="baz boo" />`,
  components:{
    myComponent
  }
}
```

`class`属性同样可以进行绑定：

```js
const myComponent = {
  template: `<p class="foo bar">Hi!</p>`
  // 渲染为 <p class="active foo bar">Hi!</p>
}
const app = {
  template: `<myComponent :class="{ active: isActive }" />`,
  components:{
    myComponent
  },
  data(){
    return{
      isActive: true
    }
  }
}
```

如果你的组件有多个根元素，你将需要指定哪个根元素来接收这个`class`。你可以通过组件的`$attrs`属性来实现指定：

```js
const myComponent = {
  template: `
    <p :class="$attrs.class">Hi!</p>
		<span>This is a child component</span>
  `
  // 渲染为
  // <p class="baz">Hi!</p>
  // <span>This is a child component</span>
}
const app = {
  template: `<myComponent class="baz" />`,
  components:{
    myComponent
  },
  data(){
    return{
      isActive: true
    }
  }
}
```

## 🔢 style

1、`style`属性支持绑定为一个对象

```js
const app = {
  template: `<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>`,
  // 渲染结果 <div style="color: red; font-size: 30px }"></div>
  data(){
    return{
      color: 'red',
      fontSize: 30
    }
  }
}
```

`style`对象内的属性可以使用小驼峰的形式进行书写：

```js
const app = {
  template: `<div :style="{ color: activeColor, 'font-size': fontSize + 'px' }"></div>`,
  // 渲染结果 <div style="color: red; font-size: 30px }"></div>
  data(){
    return{
      color: 'red',
      fontSize: 30
    }
  }
}
```

2、`style`属性绑定为一个数组

```js
const app = {
  template: `<div :style="[baseStyles, overridingStyles]"></div>`,
  // 渲染结果 <div style="color: red; font-size: 30px }"></div>
  data(){
    return{
      baseStyles: {
        color: 'red'
      },
      overridingStyles: {
         fontSize: 30
      }
    }
  }
}
```

3、`style`属性可以指定浏览器前缀

```js
const app = {
  template: `<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>`
}
```

数组仅会渲染浏览器支持的最后一个值。在这个示例中，在支持不需要特别前缀的浏览器中都会渲染为 `display: flex`。
