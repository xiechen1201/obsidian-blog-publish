---
{"dg-publish":true,"permalink":"//web/4/react18/10-ref/","dg-note-properties":{}}
---


## 过时的 API：String 类型的 Refs

Ref 是为了解决什么问题？

现代的前端框架都是响应式，开发人员不需要手动去操作 DOM 元素，只需要关心和 DOM 元素绑定响应式数据即可。

某些时候我们又不得不操作 DOM 元素，例如管理`input`的焦点、操作滚动距离等等。

早期 React 类组件中的 Ref 非常的简单，类似于 Vue：

```jsx file:App.js hl:13
import React, { Component } from 'react'

export default class App extends Component {
  clickHandle = () => {
    console.log(this);
    console.log(this.refs.inputRef);
    this.refs.inputRef.focus();
  }

  render() {
    return (
      <div>
        <input type="text" ref="inputRef"/>
        <button onClick={this.clickHandle}>聚焦</button>
      </div>
    )
  }
}
```

这样就给 `input` 创建了一个 ref 属性，通过 ref 就可以拿到 `input` 的 DOM 元素。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260701144133.png)

但是这里需要注意两点：
1. 避免使用 refs 来进行任何可以通过声明式完成的事情；
2. 这个 API 已经过时，官方建议使用回调函数或者 `createRef` API 来代替；

> [!info]
> 参阅地址：_[https://github.com/facebook/react/pull/8333#issuecomment-271648615](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Freact%2Fpull%2F8333%23issuecomment-271648615)_

## createRef API

下面是 `createRef` API 的示例：

```jsx file:App.jsx
import React, { Component } from 'react'

export default class App extends Component {

  constructor(props) {
    super();
    this.inputRef = React.createRef();
    console.log(this.inputRef); // {current: null}
  }

  clickHandle = () => {
    console.log(this.inputRef); // {current: input}
    this.inputRef.current.focus();
  }

  render() {
    return (
      <div>
        <input type="text" ref={this.inputRef}/>
        <button onClick={this.clickHandle}>聚焦</button>
      </div>
    )
  }
}
```

示例代码中，不再是通过字符串的形式创建 ref，而是通过 `React.createRef()` 创建了一个 Ref 对象，并在组件实例上新增了一个 `inputRef` 来保存这个 Ref 对象，最后和 `input` 元素进行关联。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260701144835.png)

除了在 JSX 中关联一个元素，还可以关联一个类组件，关联后就可以直接调用组件内部的方法：

```jsx file:ChildCom1.jsx
// 子组件
import React, { Component } from 'react'

export default class ChildCom1 extends Component {

    test = () => {
        console.log("这是子组件的 test 方法");
    }

    render() {
        return (
            <div>ChildCom1</div>
        )
    }
}
```

```jsx file:App.jsx hl:15,22
// 父组件
import React, { Component } from 'react';
import ChildCom1 from "./components/ChildCom1"

export default class App extends Component {

  constructor(props) {
    super();
    this.comRef = React.createRef();
  }

  clickHandle = () => {
    console.log(this);
    console.log(this.comRef); // {current: ChildCom1}
    this.comRef.current.test();
  }

  render() {
    return (
      <div>
        {/* ref 关联子组件 */}
        <ChildCom1 ref={this.comRef}/>
        <button onClick={this.clickHandle}>触发子组件方法</button>
      </div>
    )
  }
}
```

> [!warning]
> 这种行为属于反模式，应避免这样的操作。

上面的这些形式都是使用的类组件，默认情况下我们并不能直接在「函数组件」中使用 ref 属性，因为它们没有实例，但是在函数组件内部是可以使用 ref 的。

## Ref 转发

Ref 转发是一个可选的特性，也就是允许某些组件接收 ref，然后「向下传递」给子组件。

例如使用高阶组件的时候：

```jsx file:App.jsx hl:35
// App.jsx
import React, { Component } from "react";

import withLogin from "./HOC/withLog";
import ChildCom1 from "./components/ChildCom1";
const NewChild = withLogin(ChildCom1);

export default class App extends Component {
  constructor() {
    super();
    this.comRef = React.createRef();
    this.state = {
      show: true,
    };
  }

  clickHandle = () => {
    // 查看当前的 Ref 所关联的组件
    console.log(this.comRef);
  };

  render() {
    return (
      <div>
        <button
          onClick={() =>
            this.setState({
              show: !this.state.show,
            })
          }
        >
          show/hide
        </button>
        <button onClick={this.clickHandle}>触发子组件方法</button>
        {this.state.show ? <NewChild ref={this.comRef} /> : null}
      </div>
    );
  }
}
```

```jsx file:withLog.jsx
// withLog.jsx
import { Component } from "react";
import { formatDate } from "../utils/tools";

// 高阶组件是一个函数，接收一个组件作为参数
// 返回一个新的组件
function withLog(Com) {
  // 返回的新组件
  return class extends Component {
    constructor(props) {
      super(props);
      this.state = { n: 1 };
    }
    componentDidMount() {
      console.log(
        `日志：组件${Com.name}已经创建，创建时间${formatDate(
          Date.now(),
          "year-time"
        )}`
      );
    }
    componentWillUnmount() {
      console.log(
        `日志：组件${Com.name}已经销毁，销毁时间${formatDate(
          Date.now(),
          "year-time"
        )}`
      );
    }
    render() {
      return <Com {...this.props} />;
    }
  };
}

export default withLog;
```

```jsx file:ChildCom1.jsx
// ChildCom1.jsx
import React, { Component } from 'react'

export default class ChildCom1 extends Component {

    test = () => {
        console.log("这是子组件的 test 方法");
    }

    render() {
        return (
            <div>ChildCom1</div>
        )
    }
}
```

上面的示例代码中，我们使用 `withLog` 这个高阶组件来包裹 `ChildCom1` 子组件，实现了添加日志的功能，在使用高阶组件的时候我传递了一个 ref 属性，想要获取子组件中的 `test()` 方法，但实际我们会发现 Ref 关联的是高阶组件中返回的增强组件，而不是原来的子组件。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260701150444.png)

要解决这个问题就需要使用 Ref 转发。

我们可以使用官方提供的 `React.forwardRef` API：

```jsx file:withLog.jsx hl:37
import React, { Component } from "react";
import { formatDate } from "../utils/tools";

// 高阶组件是一个函数，接收一个组件作为参数
// 返回一个新的组件
function withLog(Com) {
  // 返回的新组件
  class WithLogCom extends Component {
    constructor(props) {
      super(props);
      this.state = { n: 1 };
    }
    componentDidMount() {
      console.log(
        `日志：组件${Com.name}已经创建，创建时间${formatDate(
          Date.now(),
          "year-time"
        )}`
      );
    }
    componentWillUnmount() {
      console.log(
        `日志：组件${Com.name}已经销毁，销毁时间${formatDate(
          Date.now(),
          "year-time"
        )}`
      );
    }
    render() {
      // 通过 this.props 能够拿到传递下来的 ref
      // 然后和子组件进行关联
      const {forwardedRef, ...rest} = this.props;
      return <Com ref={forwardedRef} {...rest} />;
    }
  }

  return React.forwardRef((props, ref) => {
    // 这里是关键，渲染函数会自动传入 ref，然后我们将 ref 继续往下传递
    return <WithLogCom {...props} forwardedRef={ref} />;
  });
}

export default withLog;
```

`React.forwardRef` 接收一个渲染函数，函数接收 `props` 和 `ref` 参数并返回原本我们直接返回的增强组件。

然后我们在增强组件的 `render` 方法中，通过 `this.props` 拿到 ref 继续传递给子组件。

## useRef 和 useImperativeHandle

关于 Ref 这块，最后要看一下 `useRef` 和 `useImperativeHandle` 这两个 hook。

现在的 React 都是函数组件大行其道，那么自然会涉及到函数组件和 Ref 的关联。

```jsx file:App.jsx
import React from 'react';

function App() {
  const [counter, setCounter] = React.useState(1);

  const inputRef1 = React.createRef();
  const inputRef2 = React.useRef();
  console.log("inputRef1:", inputRef1); // {current: null}
  console.log("inputRef2:", inputRef2); // {current: undefined}

  function clickHandle() {
    console.log("inputRef1:", inputRef1); // {current: input}
    console.log("inputRef2:", inputRef2); // {current: input}
    setCounter(counter + 1);
  }

  return (
    <div>
      <button onClick={clickHandle}>+1</button>
      <div>{counter}</div>
      <div>
        <input type="text" ref={inputRef1} />
      </div>
      <div>
        <input type="text" ref={inputRef2} />
      </div>
    </div>
  );
}

export default App;
```

通过示例可以看到，`React.createRef()` 和 `React.useRef()` 都可以创建 Ref，但是还是有一些区别，主要体现在：
1. `useRef`是 hooks 的一种，一般用于函数组件，而 `createRef` 一般用于类组件；
2. `useRef` 创建的 ref 对象在组件的整个生命周期内都不会改变，但是 `createRef` 创建的 Ref 对象组件每更新一次， Ref 对象都会被重新创建；

正是因为 `createRef` 创建的 Ref 每次都会重新创建，所以出现了 `useRef` 来解决这个问题。

`useRef` 还接受一个初始值，这个值和关联 DOM 的时候没有什么作用，但是「作为存储不需要变化」的全局变量非常方便。

```jsx file:App.jsx hl:1,14
import { useState, useEffect } from 'react';

function App() {
  let timer;
  const [counter, setCounter] = useState(1);

  useEffect(() => {
    timer = setInterval(() => {
      console.log('触发了');
    }, 1000);
  },[]);

  const clearTimer = () => {
    clearInterval(timer);
  }

  function clickHandle(){
    console.log(timer);
    setCounter(counter + 1);
  }

  return (
    <>
      <div>{counter}</div>
      <button onClick={clickHandle}>+1</button>
      <button onClick={clearTimer}>停止</button>
    </>)
}

export default App;
```

上面的写法存在一个问题，如果 APP 组件因 state 的变化重新 render，就会导致 timer 被重置，导致点击 `clearTimer()` 的时候是无法停止定时器的。

这个时候就可以根据 `useRef()` 在组件生命周期不会被改变的特性，将这个 timer 存储到 Ref 中。

```jsx file:App.jsx hl:4
import { useState, useEffect, useRef } from 'react';

function App() {
  let timer = useRef(null);
  const [counter, setCounter] = useState(1);

  useEffect(() => {
    timer.current = setInterval(() => {
      console.log('触发了');
    }, 1000);
  },[]);

  const clearTimer = () => {
    clearInterval(timer.current);
  }

  function clickHandle(){
    console.log(timer);
    setCounter(counter + 1);
  }

  return (
    <>
      <div>{counter}</div>
      <button onClick={clickHandle}>+1</button>
      <button onClick={clearTimer}>停止</button>
    </>)
}

export default App;
```

另外，再看下 `useImperativeHandle` 这个 hook。这个 hook 一般是配合 `React.forwardRef` 来使用的，主要作用是父组件传入 Ref 的时候，自定义要暴露给父组件的实例值。

```jsx file:App.jsx
import {useRef} from 'react';
import ChildCom1 from "./components/ChildCom1"

function App() {

  const comRef = useRef();

  function clickHandle(){
    comRef.current.click();
  }

  return (
    <div>
      <ChildCom1 ref={comRef}/>
      <button onClick={clickHandle}>触发子组件的方法</button>
    </div>
  );
}

export default App;
```

在父组件中，我们向子组件传递了一个 Ref，但是子组件实际上是一个函数组件，函数组件本身是因为挂载 Ref 的，所以需要使用 `React.forwardRef` 进行 Ref 转发，之后配合 `useImperativeHandle` 来自定义暴露给父组件的实例值。

```jsx file:ChildCom1.jsx hl:11-15
import React, { useRef, useImperativeHandle } from 'react';

function ChildCom1(props, ref) {

    const childRef = useRef();

    // 第一个是父组件传递过来的 ref
    // 第二个回调函数返回一个对象，该对象是一个映射关系
    // 映射关系中的键之后能够暴露给父组件使用
    // 映射关系中的值对应的是对应的方法
    useImperativeHandle(ref, () => ({
        click: () => {
            console.log(childRef.current);
        }
    }));

    function clickHandle() {
        console.log("这是子组件的 test 方法");
    }

    return (
        <div onClick={clickHandle} ref={childRef}>
            子组件1
        </div>
    );
}

// 需要做 ref 转发
export default React.forwardRef(ChildCom1);
```

## 使用 Ref 管理列表

如果我们的目标是关联一个 DOM 那么创建一个 Ref 对象没有问题，但是某些时候我们可能想要给一个列表的所有元素都进行 Ref 关联，那么又该如何做呢？

解决方案就是把传递给元素的 ref 属性更改为一个回调函数，这被称为「Ref 回调」，在这个回调内可以拿到每个元素的节点，然后我们自己维护一个 Map 结构进行保存即可。

```jsx file:App.jsx
const refs = useRef(new Map());

return (
  <>
    {list.map((item) => (
      <div
        key={item.id}
        ref={(node) => {
          if (node) {
            refs.current.set(item.id, node);
          } else {
            refs.current.delete(item.id);
          }
        }}
      >
        {item.name}
      </div>
    ))}
  </>
);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260701154814.png)

