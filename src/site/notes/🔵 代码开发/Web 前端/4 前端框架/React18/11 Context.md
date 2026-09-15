---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/React18/11 Context/","dg-note-properties":{}}
---


当我们的项目组件形成组件树的时候，可能存在数据需要层层传递的情况。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260729164400.png)

假设现在 subComA-1 组件的状态要传递给 subComB-2 要怎么做？
按照 props 单项数据流的逻辑，肯定是要把 subComA-1 组件的状态提取到 App 组件中，这样就可以让 subComB-2 进行消费了。
如果 subComA-1 需要更改传递进来的数据，则又需要接收从 App 组件一层层传递下来可以更改状态的方法。

Context 用于避免将数据仅为传递而逐层通过 props 传递（prop drilling）。Provider 可以把数据提供给其后代组件；它不替代状态设计，也不能让任意两个没有共同 Provider 祖先关系的组件直接共享状态。Context 意为“上下文”，所谓上下文往往指的是代码执行所需要的数据环境。

## Context 的用法

示例：

```jsx file:context/index.jsx
import { createContext } from "react";

// 创建一个上下文对象
const MyContext = createContext(null);

export { MyContext };
```

使用 `createContext` API 在组件外创建一个 Context 上下文对象。在 React 18 中，该对象带有 `Provider` 和 `Consumer`；`Provider` 用于提供值，`Consumer` 用于读取值。新编写的函数组件通常优先使用后文的 `useContext`。

```jsx file:App.jsx
import { useState } from "react";
import { MyContext } from "./context";

import ChildCom from "./ChildCom";

function App() {
	const [count, setCount] = useState(0);

	return (
		<MyContext.Provider value={{ count, setCount }}>
			<ChildCom />
		</MyContext.Provider>
	)
}

export default App;
```

在 App 组件内部，导入了创建的 Context 对象，并且使用 Provider 组件对其他的组件进行包裹，并且传递了一个 `value` 属性，这样就完成了数据的提供。

ChildCom 组件为了模拟多层组件结构，只导入了另外两个组件：

```jsx file:components/ChildCom.jsx
import React from 'react';

import ChildCom2 from "./ChildCom2"
import ChildCom3 from "./ChildCom3"

function ChildCom1() {
    return (
        <div>
            ChildCom1
            <ChildCom2/>
            <ChildCom3/>
        </div>
    );
}

export default ChildCom1;
```

在 ChildCom 的组件内部，不需要接收 App 组件传递的数据，然后再透传到 ChildCom2 和 ChildCom3 组件中。

ChildCom2 可以使用上下文对象的 Consumer 组件；ChildCom3 则使用后文展示的 `contextType`，两者都可以得到 Provider 提供的数据。

```jsx file:components/ChildCom2.jsx hl:3,7,21
import React from "react";

import { MyContext } from "../context";

function ChildCom2() {
  return (
    <MyContext.Consumer>
      {(context) => (
        <div
          style={{
            border: "1px solid",
            width: "200px",
            userSelect: "none",
          }}
          onClick={() => context.setCount((count) => count + 1)}
        >
          ChildCom2
          <div>count:{context.count}</div>
        </div>
      )}
    </MyContext.Consumer>
  );
}

export default ChildCom2;
```

**Consumer 组件内部需要提供一个「函数」作为子元素，这个函数接收当前 Context 上下文作为值，并且返回一段 JSX。**

ChildCom3 是一个类组件。类组件可以使用 _Consumer_ 来读取 Context；也可以绑定一个 `contextType` 属性订阅一个 Context，上下文值会被赋给 `this.context`。`contextType` 一次只能订阅一个 Context；需要读取多个 Context 时可使用嵌套的 `Consumer`。

```jsx file:components/ChildCom3.jsx hl:6,16,19
import React, { Component } from 'react'
import { MyContext } from "../context";

export default class ChildCom3 extends Component {
	// 自动注入 this.context
    static contextType = MyContext;

    render() {
        return (
            <div
                style={{
                    border: '1px solid',
                    width: "200px",
                    userSelect: 'none'
                }}
                onClick={() => this.context.setCount((count) => count + 2)}
            >
                ChildCom3
                <div>count:{this.context.count}</div>
            </div>
        )
    }
}
```

整个组件树的结构如下：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260729174124.png)

### displayName

在使用 React 框架开发项目的时候，通常会使用 React Developer Tools 开发者工具，那么在调试的时候我们会发现组件树的结构，默认名称就是 Context.Provider 和 Context.Consumer：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260729174442.png)

在只有一个 Context 上下文的时候还好，如果上下文对象多了就容易混乱，可以使用 `displayName` 属性来对 Context 进行显示命名。

```jsx file:context/index.js hl:5
import React from "react";

const MyContext = React.createContext(null);

MyContext.displayName = 'counter';

export { MyContext };
```

这个时候再看调试工具就可以展示我们设置的名称了：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260729174735.png)

### 默认值

在创建 Context 对象的时候可以提供默认值：

```jsx file:context/index.js hl:3-6
import React from "react";

const MyContext = React.createContext({
  count: 0,
  // 没有 Provider 时调用不会更新状态；仅用于避免示例中的点击处理函数报错。
  setCount: () => {},
});

MyContext.displayName = "counter";

export { MyContext };
```

如果希望消费的是默认值，也就不需要在 App 组件内提供 Provider 组件了：

```jsx file:App.jsx
import { MyContext } from "./context";

import ChildCom from "./ChildCom";

function App() {
	return (
		<>
			{/* 不提供 Provider 时，后代会读取 createContext 中的默认值。 */}
			<ChildCom />
		</>
	)
}

export default App;
```

当上方不存在匹配的 Provider 时，ChildCom2 和 ChildCom3 会消费默认数据。默认值是静态的；本例的 `setCount` 为空函数，因此点击不会更新 `count`。

### 多个上下文环境

Context 上下文可以存在多个，只需要在 App 组件内进行包裹即可：

```js file:context/theme.js
import { createContext } from "react";

// 默认值
const ThemeContext = createContext("light");

ThemeContext.displayName = "ThemeContext";

export { ThemeContext };
```

上面的代码中，我们又创建了一个 Context 对象，接下来在 App 组件中进行导入：

```jsx file:App.jsx
import { useState } from "react";
import { MyContext } from "./context";
import { ThemeContext } from "./context/theme";

import ChildCom from "./ChildCom";

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <MyContext.Provider value={{ count, setCount }}>
        <ThemeContext.Provider value="dark">
	        <ChildCom />
        </ThemeContext.Provider>
      </MyContext.Provider>
    </div>
  );
}

export default App;
```

然后后代组件可继续使用 Consumer（或 `contextType`、`useContext`）读取各自的 Context：

```jsx file:components/ChildCom2.jsx
import React from "react";
import { MyContext } from "../context";
import { ThemeContext } from "../context/theme";

function ChildCom2() {
  return (
    <MyContext.Consumer>
      {(context1) => (
        <ThemeContext.Consumer>
          {(context2) => (
            <div
              style={{
                border: "1px solid",
                width: "200px",
                userSelect: "none",
              }}
            >
              ChildCom2
              <div>count : {context1.count}</div>
              <div>theme : {context2}</div>
            </div>
          )}
        </ThemeContext.Consumer>
      )}
    </MyContext.Consumer>
  );
}

export default ChildCom2;
```

## Context 相关 Hook

在 React Hook 的 API 中提供了一个 `useContext` 的函数，可以让我们更加方便的使用 Context 中的数据，该 Hook 接收一个 Context 上下文对象，并返回上下文对象的数据。

例如我再创建一个 languageContext 上下文对象：

```js file:context/language.js
import { createContext } from "react";

const LanguageContext = createContext("en");

LanguageContext.displayName = "LanguageContext";

export { LanguageContext };
```

然后在 App 组件内部进行导入：

```jsx file:App.jsx hl:3,12,14
...

import { LanguageContext } from "./context/language";

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <MyContext.Provider value={{ count, setCount }}>
        <ThemeContext.Provider value="dark">
	        <LanguageContext.Provider value='en'>
		        <ChildCom />
		    </LanguageContext.Provider>
        </ThemeContext.Provider>
      </MyContext.Provider>
    </div>
  );
}

export default App;
```

然后在 ChildCom2 组件内部使用 `useContext` 消费 LanguageContext 对象：

```jsx file:components/ChildCom2.jsx
import { useContext } from "react";
import { MyContext } from "../context";
import { ThemeContext } from "../context/theme";
import { LanguageContext } from "../context/language";

function ChildCom2() {

 const context3 = useContext(LanguageContext);
 console.log("🚀 ~ ChildCom ~ context3:", context3);

  return (
    <>
      <MyContext.Consumer>
        {(context1) => (
          <ThemeContext.Consumer>
            {(context2) => (
              <div
                style={{
                  border: "1px solid",
                  width: "200px",
                  userSelect: "none",
                }}
              >
                ChildCom2
                <div>count : {context1.count}</div>
                <div>theme : {context2}</div>
                <div>检测到语言：{context3}</div>
              </div>
            )}
          </ThemeContext.Consumer>
        )}
      </MyContext.Consumer>
    </>
  );
}

export default ChildCom2;
```

`useContext(MyContext)`相当于类组件中的 `static contextType = MyContext` 或者 `<MyContext.Consumer>`，但是我们仍然需要在上层组件树中使用 `<MyContext.Provider>` 来为下层组件提供 context。
