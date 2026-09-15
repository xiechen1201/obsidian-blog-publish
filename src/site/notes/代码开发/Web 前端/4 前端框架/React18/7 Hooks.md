---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/7 Hooks/","dg-note-properties":{}}
---

## 🔢 简介

Hook 是 React16.8 的新增特性，它可以让你在不写「类组件」的情况下使用`state`以及其他的 React 特性。

Hook 解决了一些问题：

1、告别令人疑惑的生命周期函数。

```jsx
import React from "react";

class App extends React.Component {

  constructor() {
    super();
    this.state = {
      count : 0
    }
  }

  componentDidMount(){
    document.title = `你点击了${this.state.count}次`;
  }

  componentDidUpdate(){
    document.title = `你点击了${this.state.count}次`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    )
  }
}

export default App;
```

以上代码中，我们在两个生命周期函数中书写了两次相同的代码。

2、告别类组件中烦人的`this`。

在类组件中，会存在`this`指向的问题，例如在事件处理函数中，不能直接通过`this`获取组件的实例，需要修改`this`的指向。

3、告别繁重的类组件，让开发者回归熟悉的「函数式」编程中去。

4、一些不相关的代码都写在一个钩子函数中，容易产生一些 Bug。

Hooks 的出现，标志着整个 React 思想的转变，从「面向对象编程」的思想转化为了「函数式编程」的思想。

因为是函数式编程，所以出现了一些不太熟悉的名词，例如：纯函数、副作用、柯里化函数、高阶函数等。

Hooks 本质上就是 JavaScript 的函数，但是使用的时候有两个额外的规则：

- 只能在函数的最外层调用 Hook，不能在循环、条件判断或者子函数中调用；
- 只能在 React 的函数组件中调用 Hook，不能在其他 JavaScript 函数中调用；

React 中内置了非常多的 Hook，我们先学两个简单的！

## 🔢 useState()

[useState – React 中文文档](https://zh-hans.react.dev/reference/react/useState)

- 该 Hook 用来设置组件的数据状态。
- 该 Hook 接受一个初始化的数据作为参数。
- 该 Hook 执行后返回一个数组，第一项是`state`的值，第二项是更改`state`对应的函数。

```jsx
import { useState } from 'react';

function App() {
  // num 表示数据
  // setNum 表示更改数据的方法
  let [num, setNum] = useState(0);

  function onClickBtn() {
		// 传递一个数据来更改 num 的值
    setNum(++num);
  }

  return (
    <>
      <p>{num}</p>
      <button onClick={onClickBtn}>Add Num</button>
    </>
  );
}
export default App;
```

可以创建多个`state`：

```jsx
import { useState } from 'react';

function App(props) {

	let [age, setAge] = useState(18);
	const [fruit, setFruit] = useState('banana');
	const [todos, setTodos] = useState([{ text: '学习 Hook' }]);

	function clickhandle(){
		setAge(++age);
	}

	return (
		<div>
			<div>年龄：{age}</div>
			<div>水果：{fruit}</div>
			<div>待办事项：{todos[0].text}</div>
			<button onClick={clickhandle}>+1</button>
		</div>
	);
}

export default App;
```

## 🔢 useEffect()

该 Hook 用来创建一个副作用函数。

什么是「副作用函数」？

要解释副作用函数就必须搞清楚什么是「非副作用函数（即纯函数）」？

纯函数：一个确切的参数在你的函数调用后，一定能得到一个符合预期确切的值。

```jsx
// 放在函数式编程中，这就一个纯函数。
function test(x) {
  return x * 2;
}
```

如果函数内存在副作用，那么该函数则不是纯函数，所谓副作用函数就是指函数的结果是不可预期的！

常见的副作用：发送网络请求、添加监听事件、修改 DOM、读取文件等。

### > 基本使用

之前我们都是将这些副作用的操作放在生命周期函数中，但是现在提供了这个 Hook 专门让我们书写副作用。

==`useEffect()`会在`render()`渲染完成后执行，==类似于 mount 和 update 的结合，所以该 Hook 会在`state`更改后再次执行！

```jsx
import { useEffect } from 'react';

export default function App() {
	let [count, setCount] = useState(0);

	useEffect(() => {
		// 书写你要执行的副作用，会在 render 完成后执行，类似于 mount 和 update 的结合
		// useEffect 会在每次状态更改后都执行
		console.log('useEffect');

		// 每次执行 onClickBtn() 都会执行这里的副作用
		document.title = '你点击了' + count + '次';
	});

	function onClickBtn() {
		setCount(count + 1);
	}

	return (
		<>
			<div>{count}</div>
			<button onClick={onClickBtn}>Add Num</button>
		</>
	);
}
```

不出意外的话，当你每次点击按钮的时候都会去更新`state`，当`state`更新的时候就会触发副作用函数的执行！

### > 清理函数

如果我们在副作用内加一个定时器，每次延时打印想要的结果：

```jsx
import { useEffect } from 'react';

export default function App() {
	let [count, setCount] = useState(0);

	useEffect(() => {
		setInterval(() => {
			console.log('useEffect');
		}, 1000);
	});

	function onClickBtn() {
		setCount(count + 1);
	}

	return (
		<>
			<div>{count}</div>
			<button onClick={onClickBtn}>Add Num</button>
		</>
	);
}
```

效果如下：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/1713169472347-70d2bf38-94fc-40ea-b01d-045c55b043c0.gif)

产生这样的效果是因为我们每次更改`state`都会触发副作用函数的重新执行，而每次执行都会创建一个定时器去打印`useEffect`字符串，这就导致了`useEffect`被不可控的打印。

要解决这个问题，==我们可以给`useEffect()`返回一个函数，该函数被称为「清理函数」（主要是用来做一些清理的操作），==该函数会在`render()`执行之后，下一次`useEffect()`执行之前执行，所以可以用来清理上一次副作用内的程序：

```jsx
import { useEffect } from "react";

export default function App() {
  let [count, setCount] = useState(0);

  useEffect(() => {
    console.log("useEffect calling");

    return () => {
      console.log("useEffect clear");
    };
  });

  function onClickBtn() {
    setCount(count + 1);
  }

  return (
    <>
      <div>{count}</div>
      <button onClick={onClickBtn}>Add Num</button>
    </>
  );
}
```

可以看到，当每次点击按钮的时候，都会先执行「清理函数」再执行副作用函数！因此，我们可以通过这个函数来清理上一次副作用函数内的定时器：

```jsx hl:11-14
import { useEffect } from "react";

export default function App() {
  let [count, setCount] = useState(0);

  useEffect(() => {
    const stopTimer = setInterval(() => {
      console.log("useEffect");
    }, 1000);

    return () => {
      // 清理定时器
      clearInterval(stopTimer);
    };
  });

  function onClickBtn() {
    setCount(count + 1);
  }

  return (
    <>
      <div>{count}</div>
      <button onClick={onClickBtn}>Add Num</button>
    </>
  );
}

```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/1713170767733-6ce5bdc2-a4e2-4fe0-8506-476be7ec188d.gif)

这样每次点击按钮更改`state`的时候都会清理上一次的定时器，并启动一个新的定时器啦！

### > 副作用依赖

目前我们的副作用函数，会在每次渲染完成后和更改`state`后重新执行，`useEffect()`还可以传递第二个参数，表示该副作用函数依赖的数据，只有当依赖的数据发生变化的时候才会触发执行。

```jsx
import { useState, useEffect } from 'react';

function App() {

  let [count1, setCount1] = useState(0);
  let [count2, setCount2] = useState(0);
  let [count3, setCount3] = useState(0);

  useEffect(()=>{
    console.log("执行副作用函数");
  });

  return (
    <div>
      <div>count1:{count1}</div>
      <div>count2:{count2}</div>
      <div>count3:{count3}</div>
      <button onClick={()=>setCount1(++count1)}>+1</button>
      <button onClick={()=>setCount2(++count2)}>+1</button>
      <button onClick={()=>setCount3(++count3)}>+1</button>
    </div>
  );
}

export default App;
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/1713171159606-475c1c60-3489-4fdb-8b74-792a7d1202e8.gif)

如图所示，当我点击任意一个按钮的时候都会更改对应的`state`，每当`state`更改的时候都会触发`useEffect()`函数的执行，如果我想要实现只有更改`count1`的时候再触发更新就需要穿`useEffct()`传递依赖的数据源：

```jsx
useEffect(() => {
	console.log('执行副作用函数');
}, [count1]);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/1713171335971-9d2e5582-d832-402d-bd08-8e860b3a5ba3.gif)

如图所示，现在我点击第二个和第三个按钮更改`state`后就不会再执行`useEffct()`函数的执行了。

如果你只想要`useEffct()`在初始化的时候执行一次，后面更改`state`的时候都不要再更新，那么传递第二个参数为空即可：

```jsx
// 只会在初始化的时候执行一次
useEffect(() => {
	fetchData().then((res) => {
		setCount(res);
	});
}, []);
```

## 🔢 自定义的 Hook

除了使用官方写好的 Hooks，我们还可以自定义 Hook，自定义的 Hook 本质上就是函数，但是和普通的函数有一些区别，主要体现在两个方面：
1. 自定义的 Hook 能够调用`useState`、`useEffct`等，而普通函数则不能。
2. 自定义的 Hook 需要以 use 开头，普通函数则没有这个限制。使用 use 开头并不是一种语法而是一种约定。

示例：

```jsx
import { useState } from 'react';
import useMyBook from "./useMyBook"

function App() {

  const {bookName, setBookName} = useMyBook();
  const [value, setValue] = useState("");

  function changeHandle(e){
    setValue(e.target.value);
  }

  function clickHandle(){
    setBookName(value);
  }

  return (
    <div>
      <div>{bookName}</div>
      <input type="text" value={value} onChange={changeHandle}/>
      <button onClick={clickHandle}>确定</button>
    </div>
  )

}

export default App;
```

```jsx
import { useState } from "react";

function useMyBook(){
    const [bookName, setBookName] = useState("React 学习");
    return {
        bookName, setBookName
    }
}

export default useMyBook;
```
