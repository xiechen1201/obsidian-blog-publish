---
{"dg-publish":true,"permalink":"//web/4/react18/2-jsx/","dg-note-properties":{}}
---

## JSX 基础语法

在 React 中使用 JSX

语法来描述视图：

```jsx
function App() {
	return (
		<div>Hello React~</div>
	);
}
```

还可以把类似于 HTML 的代码单独提取出来：

```jsx
function App() {
	const ele = <div>Hello React~</div>
	return (
		ele
	);
}
```

JSX 只是长得像 HTML，但其实是 JavaScript 的语法。JSX 语法乍一看比较像模版语言，但其实他完全是在 JavaScript 内部实现的。

JSX 语法的规则：
- 根元素只能有一个；

```jsx
// 使用 <div> 或者空的 <> 进行包裹
function App() {
	return (
		<>
			<ul>
				<li>1</li>
				<li>2</li>
				<li>3</li>
			</ul>
			<ul>
				<li>1</li>
				<li>2</li>
				<li>3</li>
			</ul>
		</>
	)
}
```

- 在 JSX 内部使用 JS 表达式，表达式必须写在`{}`内部；
- `{}`只能书写 JS 表达式，不能书写语句；
- 给元素绑定类名不能使用`class`，而是使用`className`代替；
- 注释需要写在`{}`内部；

```jsx
function App() {
	return (
		<ul>
			<li>1</li>
			<li>2</li>
			{/* 正确 */}
			<li>{ 3 + 1 }</li>
			<li className={ true ? 'two' : 'three' }>2</li>
			{/* 错误 */}
			<li>{ if(true){ 1 }}</li>
		</ul>
	);
}
```

- JSX 允许在模版中插入数组，数组会自动展开所有成员；

```jsx
function App() {
  const arr = [<p key="1">Hello</p>, <p key="2">world</p>, <p key="3">hhh</p>];
  return (
    <>
      <ul
        style={{
          color: "red",
          fontSize: "16px",
        }}
      >
        <li>1</li>
        {arr}
      </ul>
    </>
  );
}
```

## createElement() 方法

JSX 是 JS 的一种语法拓展，Babel 等编译工具会把 JSX 转译为一个名为`React.createElement()`的函数调用。

```jsx
React.createElement(type, [props], [...children]);
```

> [!info]
> 参数说明：
> - type：创建 React 元素类型，可以是标签名字符串、React 组件；
> - props：可选，React 元素的属性；
> - children：可选，React 元素的子元素；

使用 JSX 语法和`createElement()`方法对比，效果是等价的：

```jsx
const element1 = <h1 className='greeting'> Hello, world! </h1>;
const element2 = React.createElement(
  "h1",
  { className: "greeting" },
  "Hello, world!"
);
```

<img src="./_assets/1712040390799-c8169e9f-7aad-4541-bd48-1488c2100dc8.png" width="1127" title="" crop="0,0,1,1" id="u0e4089d0" class="ne-image">

这些对象被称为“React 元素”，他们描述了你希望在页面上看到的内容。

所以也可以把 JSX 理解为是`React.createElement()`方法的语法糖。
