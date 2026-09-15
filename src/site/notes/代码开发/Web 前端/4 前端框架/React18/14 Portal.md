---
{"dg-publish":true,"permalink":"//web/4/react18/14-portal/","dg-note-properties":{}}
---

Portals 意为传送门，它要做的事情实际上和传送门确实很类似，根据官方的解释：

>  Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。

语法：

```js
ReactDOM.createPortal(child, container)
```

`child` 表示任何一个 React 的子元素，`container` 表示一个 DOM 元素。

## 什么场景需要 Portals?

例如：

```jsx file:App.jsx
import { useState } from "react";
import Modal from "./components/Modal"

function App() {
  const [isShow, setIsShow] = useState(false);
  
  return (
    <div>
      <h1>App组件</h1>
      <button onClick={()=>setIsShow(!isShow)}>显示/隐藏</button>
      {isShow ? <Modal /> : null}
    </div>
  );
}

export default App;
```

```jsx file:Modal.jsx
function Modal() {
    return (
        <div style={{
            width : "450px",
            height : "250px",
            border : "1px solid",
            position : "absolute",
            left : "calc(50% - 225px)",
            top : "calc(50% - 125px)",
            textAlign : "center",
            lineHeight : "250px"
        }}>
	        模态框
        </div>
    )
}

export default Modal;
```

上面的代码中，我们在 App 组件中通过 button 来控制 Modal 模态框的展示/隐藏，功能上倒是没有但是，但是在 DOM 的层级上就显的不太合适。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260902151353.png)
并且如果父元素设置了某些样式可能也会影响到模态框：

```jsx file:App.jsx hl:2
<div style={{
  position: "relative"
}}>
  <h1>App组件</h1>
  <button onClick={() => setIsShow(!isShow)}>显示/隐藏</button>
  {isShow ? <Modal /> : null}
</div>
```

这个时候就可以使用 Portal 来解决这个问题。

## 如何使用？

我们只需要调用 `createPortal()` 方法并指定渲染到哪个元素上即可。需要注意的是这个方法适合 React 渲染相关，因此它需要从 react-dom 中导入。

```jsx file:Modal.jsx
import {createPortal} from 'react-dom';

function Modal() {
    return createPortal((<div style={{
        width : "450px",
        height : "250px",
        border : "1px solid",
        position : "absolute",
        left : "calc(50% - 225px)",
        top : "calc(50% - 125px)",
        textAlign : "center",
        lineHeight : "250px"
    }}>模态框</div>), document.getElementById("modal"))
}

export default Modal;
```

上面的代码中，我们将要渲染的内容作为第一个参数传入，将要挂载的节点作为第二个参数传入。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260902151901.png)
可以看到，现在模态框就被渲染到根元素下面了，同时 App 组件中的样式也不会影响到 Modal。

## Portal 的事件冒泡

上面的示例中，虽然模态框被渲染到了根元素下面，但是 Modal 的事件冒泡依旧是按照在组件中的结构进行冒泡。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260902152147.png)
从调试工具中可以看到，Modal 依旧属于 App 下的子组件。

使用一个示例来佐证：

```jsx file:App.jsx
import { useState } from "react";
import Modal from "./components/Modal"

function App() {

  const [isShow, setIsShow] = useState(false);
  
  return (
    <div style={{
      position: "relative"
    }} onClick={()=>console.log("App 组件被点击了")}>
      <h1>App组件</h1>
      <button onClick={() => setIsShow(!isShow)}>显示/隐藏</button>
      {isShow ? <Modal /> : null}
    </div>
  );
}

export default App;
```

此时，我们点击 Modal 依旧是可以触发 App 的中的 click 事件的。

正如官方文档所说：

> 尽管 portal 可以被放置在 DOM 树中的任何地方，但在任何其他方面，其行为和普通的 React 子节点行为一致。由于 portal 仍存在于 React 树， 且与 DOM 树中的位置无关，那么无论其子节点是否是 portal，像 context 这样的功能特性都是不变的。
> 
> 这包含事件冒泡。一个从 portal 内部触发的事件会一直冒泡至包含 React 树的祖先，即便这些元素并不是 DOM 树中的祖先。