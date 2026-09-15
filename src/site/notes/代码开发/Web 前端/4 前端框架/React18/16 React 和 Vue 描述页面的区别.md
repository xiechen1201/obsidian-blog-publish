---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/16 React 和 Vue 描述页面的区别/","dg-note-properties":{}}
---

比较简陋的一个回答是：Vue 使用的是 `template` 描述页面，而 React 使用的是 JSX。
这也是目前比较主流描述 UI 的两种方案。

## JSX 的历史来源

JSX 最早源自于 React 团队在 React 中提供的一种类似于 XML 的 ES 语法糖：

```jsx
const element = <h1>Hello</h1>
```

经过 Babel 编译工具编译后就会变成：

```js
// React v17 之前
var element = React.createElement("h1", null, "Hello");

// React v17 之后
var jsxRuntime = require("react/jsx-runtime");
var element = jsxRuntime.jsx("h1", {children: "Hello"});
```

无论是 v17 之前还是之后，只要是执行了代码就会得到一个对象：

```js
{
  "type": "h1",
  "key": null,
  "ref": null,
  "props": {
    "children": "Hello"
  },
  "_owner": null,
  "_store": {}
}
```

这就是虚拟 DOM。

React 团队认为 UI 本质上和逻辑是存在耦合的，例如：
- 在 UI 上绑定事件；
- 数据变化后通过 JS 去改变 UI 的样式或者结构；

前端开发中 JS 使用的最多，所以 React 团队考虑屏蔽 HTML，使用 JS 来描述整个 UI，这样的话逻辑和视图就更加紧密了，最终设计出类 XML 的 JS 语法糖。

由于 JSX 是 JS 的语法糖，可以灵活的使用 JS 进行组合：
- 在  if 或者 for 中使用 JSX；
- 将 JSX 赋值给变量；
- 把 JSX 当作参数进行传递，也可以在一个函数中返回一段 JSX；

```jsx
function App({isLoading}){
  if(isLoading){
    return <h1>loading...</h1>
  }
  return <h1>Hello World</h1>;
}
```

## template 的历史来源

模版的历史要从后端说起。

早期前后端未分离的时候，最流行的方案就是模版引擎，模版引擎可以看作是在正常的 HTML 上面进行“挖坑”，“挖坑”之后服务器端就会将数据填充到“坑”的模版里面，生成对应的 HTML 页面给客户端。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260907092313.png)
当时的前端主要工作就是 HTML、CSS 和一些简单的 JS 效果，写好的 HTML 是不能直接使用的，需要和后端确定使用的是哪一个模版引擎，接下来将自己写好的 HTML 按照对应的引擎语法进行“挖坑”。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260907092526.png)
下面是几个常见的模版引擎：
- Java：JSP、Thymeleaf
- PHP：Smarty、Twig
- NodeJS：Jade、Ejs

例如 Ejs 的语法：

```html
<h1>
    <%=title %>
</h1>
<ul>
    <% for (var i=0; i<supplies.length; i++) { %>
    <li>
        <a href='supplies/<%=supplies[i] %>'>
            <%= supplies[i] %>
        </a>
    </li>
    <% } %>
</ul>
```

这些引擎的模版语法和 Vue 里面的模版非常的相似。随着前后端分离，这些模版引擎已经不再使用了。

最后做一个总结，虽然前端中存在两种方式来描述 UI，但是它们的出发点是不一样的。
==模版的出发点是：既然前端使用 HTML 来描述 UI，那么就拓展 HTML，让 HTML  能够描述一定的逻辑，也就是“从 UI 出发，拓展 UI，在 UI 中描述逻辑”。==
==JSX 的出发点是：既然前端使用 JS 来描述逻辑，那么就拓展 JS，让 JS 描述 UI，也就是“从逻辑出发，拓展逻辑，描述 UI”。==

这两个虽然都可以描述 UI，但是思路和方向是完全不同的，从而造成了整个框架上面也是不一致的。
