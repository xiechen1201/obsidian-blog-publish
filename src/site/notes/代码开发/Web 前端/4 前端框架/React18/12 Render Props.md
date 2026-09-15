---
{"dg-publish":true,"permalink":"//web/4/react18/12-render-props/","dg-note-properties":{}}
---

在 React 中，代码复用的基本单位就是组件，可是如何组件中也存在重复的代码怎么办呢？

这个时候就需要将公共的代码提取出来，公共的这部分逻辑被称为“横切关注点”。

React 中有两种方式来进行横切关注点的抽离：
- HOC 高阶组件
- Render Props

> [!info]
> Render Props 并不是什么新技术，而是将一个渲染函数作为 prop 传递给组件。

## 如何使用？

假设有这么一个示例：

```jsx App.jsx
import ChildCom1 from "./components/ChildCom1";
import ChildCom2 from "./components/ChildCom2";

function App() {
  return (
    <div
      style={{
        display: "flex",
        width: "850px",
      }}
    >
      <ChildCom1 />
      <ChildCom2 />
    </div>
  );
}

export default App;
```

```jsx file:components/ChildCom1.jsx
// 这个文件内的逻辑也很简单，就是监听鼠标的移动，然后在页面中打印出来

import { useState } from "react";

function ChildCom1() {
  const [points, setPoints] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPoints({ x: e.clientX, y: e.clientY });
  };

  return (
    <div
      style={{
        width: "400px",
        height: "400px",
        backgroundColor: "#999",
      }}
      onMouseMove={handleMouseMove}
    >
      <h1>移动鼠标</h1>
      <p>
        当前鼠标的当前位置：{points.x}, {points.y}
      </p>
    </div>
  );
}

export default ChildCom1;
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260729195324.png)

```jsx components/ChildCom2.jsx
// 这个文件内同样监听鼠标的移动，但是这个页面会有一个小圆球跟随鼠标的移动

import { useState } from "react";

function ChildCom2() {
  const [points, setPoints] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPoints({ x: e.clientX, y: e.clientY });
  };

  return (
    <div
      style={{
        position: "relative",
        width: "400px",
        height: "400px",
        border: "1px solid #333",
      }}
      onMouseMove={handleMouseMove}
    >
      <h1>移动鼠标</h1>
      <div
        style={{
          position: "absolute",
          width: "10px",
          height: "10px",
          backgroundColor: "#333",
          borderRadius: "50%",
          left: points.x - 400 - 15,
          top: points.y - 15,
        }}
      ></div>
    </div>
  );
}

export default ChildCom2;
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260729195406.png)

基于上面的示例可以发现，两个组件其实都存在一样的逻辑，只是视图展示不一致。

基于此，我们可以抽离一个公共的组件，然后给这个公共组件传入不同的视图即可。

```jsx file:components/MouseMove.jsx
import { useState } from "react";

// 负责公共的逻辑
function MouseMove(props) {
  const [points, setPoints] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPoints({ x: e.clientX, y: e.clientY });
  };

// 接收一个 render 函数，然后调用函数，并且传递状态和函数
  return <div>{props?.render({ points, handleMouseMove })}</div>;
}

export default MouseMove;
```

然后在 App 组件中进行导入：

```jsx file:App.jsx
import MouseMove from "./components/MouseMove";
import ChildCom1 from "./components/ChildCom1";
import ChildCom2 from "./components/ChildCom2";

function App() {
  return (
    <div
      style={{
        display: "flex",
        width: "850px",
      }}
    >
	    {/* 传递一个渲染函数属性用于页面渲染 */}
	    {/* 把 props 传递给组件 */}
        <MouseMove render={(props) => <ChildCom1 {...props} />} />
        <MouseMove render={(props) => <ChildCom2 {...props} />} />    
    </div>
  );
}

export default App;
```

既然把逻辑都抽离到 MouseMove 组件中了，那么 ChildCom1 和 ChildCom2 组件就需要把逻辑删除，只保留视图部分即可。

```jsx file:components/ChildCom1.jsx
// 这个文件内的逻辑也很简单，就是监听鼠标的移动，然后在页面中打印出来
function ChildCom1(props) {
  return (
    <div
      style={{
        width: "400px",
        height: "400px",
        backgroundColor: "#999"
      }}
      onMouseMove={props.handleMouseMove}>
      <h1>移动鼠标</h1>
      <p>
        当前鼠标的当前位置：{props.points.x}, {props.points.y}
      </p>
    </div>
  );
}

export default ChildCom1;
```

```jsx components/ChildCom2.jsx
function ChildCom2(props) {
  return (
   <div
      style={{
        position: "relative",
        width: "400px",
        height: "400px",
        border: "1px solid #333"
      }}
      onMouseMove={props.handleMouseMove}>
      <h1>移动鼠标</h1>
      <div
        style={{
          position: "absolute",
          width: "10px",
          height: "10px",
          backgroundColor: "#333",
          borderRadius: "50%",
          left: props.points.x - 400 - 15,
          top: props.points.y - 15
        }}></div>
    </div>
  );
}

export default ChildCom2;
```

这样就把 ChildCom1 和 ChildCom2 的逻辑抽离出去了，这两个组件只关心自己的视图渲染即可。

> [!info] 
> 当然 `render` 属性不是必须这么命名，只要是符合语义化的均可。

## 什么时候使用 Render Props？

Render Props 和 HOC 都属于关注点横切分离，那么他们各自都在什么场景下使用呢？

一般来说：
- 如果多个组件的内部逻辑完全一致，仅仅是渲染的视图不同，就适合使用 Render Props；
- 如果多个组件仅仅一部分逻辑一致，同时又都存在各自的逻辑，那么就适合使用 HOC 高阶组件；
