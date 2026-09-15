---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/18 Vue 就地更新策略/","dg-note-properties":{}}
---

在 Vue 文档中关于`v-for`指南模块有这么一段话：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676615327189-07f1df08-38da-4b04-9d01-cecc8b928722.png)

那什么是就地更新策略呢？我们先来写一个案例，通过案例来介绍。

```js
const app = {
  template: `
    <ul>
    	<li v-for="(item, index) of list">
      	<span>{{ item.value }}</span>
        <button @click="delteItem(index)">Del</button>
      </li>
    </ul>`,
  data(){
    return{
      list: [
        {
          id: 1,
          value: "item-1"
        },
        {
          id: 2,
          value: "item-2"
        },
        {
          id: 3,
          value: "item-3"
        }
      ]
    }
  },
  methods: {
    delteItem(index) {
      this.list.splice(index, 1);
    }
  }
}
```

以上代码，实现了一个简单的列表，然后通过按钮可以进行删除某项，效果如下：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676615791447-19c162c2-3004-4504-80b9-6228b7272171.png)

## 点击删除 DOM 的变化

当我们删除`item-2`的时候看看会发生什么？

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676615965074-480c8341-5723-4f81-b4da-cabdb689b2c7.gif)

从上面的动态图中我们可以看到：我们删除的是`item-2`的`li`，第二个`li`里面的内容变成了`item-3`的内容，实际上删除的是第三个`li`（浏览器中紫色闪烁表示`dom`的内容发生了变化）。

这就是 Vue 的就地更新策略，回头看 Vue 文档的那句话：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676616226536-0c4cfe02-57fa-4042-bb79-93348f4f6ada.png)

Vue 将使用一种最小化元素移动的算法，并尽可能地就地更新/复用相同类型的元素。

也就是说默认的情况下 Vue 会尽量使用已经存在的 DOM 元素，直接在已有的 DOM 上进行复用修改，这样可以带来一定性能上的提升。

<br/>

看一个更明显的案例：

```js
const app = {
  template: `
  <div>
  	<div v-if="isLogin">
        <span>欢迎</span>
        <a href="javascript:;" @click="isLogin = false">xiechen</a>
    </div>
    <div v-else>
        <a href="javascript:;" @click="isLogin = true">登录</a>
        <a href="javascript:;">注册</a>
    </div>
  </div>`,
  data(){
    return{
      isLogin: false
    }
  }
}
```

以上代码，两个模块里都有`a`标签，按照我们的理解当条件发生变化的时候，`div`都会重新渲染，实际上 Vue 会进行就地更新策略。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676616817129-07581e81-49e2-4fa7-b6eb-39b379250fc8.gif)

可以很明显的看到第二个`a`标签一直在闪烁，也就是内容一直在更新而不是销毁重新创建的`a`标签！！！

## 就地更新的缺陷

再次回到 Vue 文档的那句话：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676617070346-6f8c971c-938d-412a-bcc3-47dce45a5543.png)

这又是什么意思呢？同样的我们还是用动态图要进行演示，我们把第一个案例进行改造，给每个`li`都新增一个`input`输入框：

```html
<ul>
  <li v-for="(item, index) of list">
    <span>{{ item.value }}</span>
    <input type="text" />
    <button @click="delteItem(index)">Del</button>
  </li>
</ul>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676617358836-85a136f0-e8a4-4e72-bd49-931865a4ab5c.gif)

实际的效果有点让你疑惑，我明明删除的是`item-2`，为什么`item-3`的输入框内容却发生了变化？

这其实就是「就地更新」的缺陷！！！

因为`input`输入框是一个临时的状态，Vue 无法判断节点的`value`到底有什么用，所以在删除`item2`的时元素会进行复用，`input`里的值也是会被保留的，实际删除的是第三个`li`及第三个`li`内的`input`。

<br/>

## 如何进行解决？

现在的问题就是 Vue 不会根据最新的顺序去更新 DOM，而是用已有的 DOM 进行属性的修改。

在这种情况下，我们需要给`li`标签绑定一个`key`属性，这样 Vue 就会根据最新的数据对 DOM 进行调整，而它会基于`key`的变化重新排列元素顺序，并且会移除`key`不存在的元素。

可以简单认为`key`是给每一个 DOM 节点一个唯一标识，这样 Vue 就不会启用就地更新了。

```html
<ul>
  <li v-for="(item, index) of list" :key="item.id">
    <span>{{ item.value }}</span>
    <input type="text" />
    <button @click="delteItem(index)">Del</button>
  </li>
</ul>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676618574766-9ce72955-a81a-4660-b42e-1385e83e23f2.gif)

从图片中可以看到，加上`key`属性后`item-2`删除后，`item-2`的`input`也被删除啦。

## key 的作用

简单说`key`的作用主要是为了更高效的对比虚拟 DOM 中每个节点是否是相同节点；

举个简单的例子：三胞胎战成一排，你怎么知道谁是老大？如果老大皮了一下子，和老三换了一下位置，你又如何区分出来？给他们挂个牌牌，写上老大、老二、老三。这样就不会认错了，`key`就是这个作用。

所以在使用`key`属性的时候需要注意，`key`属性必须是唯一的，不变的！当数据更新的时候，Vue 可以通过`key`属性去确认元素，同时确认子元素是否要进行更新。

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/_assets/1676618864145-c6388373-3224-4918-abd4-8df2bc3672b1.png)
