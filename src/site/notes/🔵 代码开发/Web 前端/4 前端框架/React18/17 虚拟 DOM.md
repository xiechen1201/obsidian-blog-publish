---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/4 前端框架/React18/17 虚拟 DOM/","dg-note-properties":{}}
---

什么是虚拟 DOM？
一个简单的回答是：虚拟 DOM 本质上就是一个普通的 JS 对象，用来描述视图的界面结构。

虚拟 DOM 最早也是 React 团队提出的，==Virtual DOM 是一种编程概念。在这个概念中，UI 以一种理想化的，或者说虚拟的表现形式被保存于内存中。==

也就说，只要我们使用一种方式，能够将真实的 DOM 层次结构描述出来，那么就是一个虚拟 DOM。
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260907093831.png)
（使用汉堡、薯条、牛排来描述 DOM 的层级结构，这也可以认为是虚拟 DOM。）

React 团队中使用的是 JS 来描述 DOM 的结构，因此很多人认为 JS 对象和虚拟 DOM 是等号，这种理解是错误的。
==虚拟 DOM 和 JS 对象的关系：前者是一种思想，后者是实现这个思想的具体方式。==

## 为什么需要虚拟 DOM？

虚拟 DOM 主要有两个优势：
- 相较于 DOM 的体积和速度优势；
- 多平台的渲染抽象能力；

### 体积优势

首先需要明确，JS 层面的计算速度要比 DOM 层面的计算要快：
- DOM 对象在被浏览器渲染出来之前，浏览器还存在很多的工作要处理（[[🔵 代码开发/Web 前端/5 浏览器与网络/浏览器/01 浏览器的渲染流程\|01 浏览器的渲染流程]]）；
- DOM 对象上面的属性非常多；
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260907094524.png)

因此，操作 JS 对象和操作 DOM 对象的时间是完全不同的。JS 层面的计算速度要高于 DOM 层面的计算速度。

这里抛出一个问题：虽然使用了 JS对象来描述 UI，但是最终渲染不还是使用原生 DOM API 来操作 DOM 吗？

虚拟 DOM 在第一次渲染页面的时候，并没有什么优势，速度肯定要比直接操作原生 DOM API 要慢一些，虚拟 DOM 的优势在于更新阶段。

根据 React 团队的研究，渲染页面的时候，相比原生的 DOM API，开发者更喜欢使用 `innerHTML` 这个属性进行直接更改。

```js
let newP = document.createElement("p");
let newContent = document.createTextNode("this is a test");
newP.appendChild(newContent);
document.body.appendChild(newP);

// 直接更改
document.body.innerHTML = `
	<p>
		this is a test
	</p>
`;
```

使用 `innerHTML` 的时候就涉及到两个层面的计算：
- JS 层面：解析字符串；
- DOM 层面：创建对应的 DOM 节点；

和虚拟 DOM 进行对比：

|          | innerHTML    | 虚拟 DOM       |
| -------- | ------------ | ------------ |
| JS 层面计算  | 解析字符串        | 创建 JS 对象     |
| DOM 层面计算 | 创建对应的 DOM 节点 | 创建对应的 DOM 节点 |
虚拟 DOM 的优势在于更新阶段。`innerHTML` 进行更新的时候需要全部重新赋值，这意味着之前创建的 DOM 节点需要全部销毁，然后重新创建。
但是虚拟 DOM 只需要更新必要的 DOM 节点即可。

|          | innerHTML    | 虚拟 DOM       |
| -------- | ------------ | ------------ |
| JS 层面计算  | 解析字符串        | 创建 JS 对象     |
| DOM 层面计算 | 销毁之前的 DOM 节点 | 修改必要的 DOM 节点 |
| DOM 层面计算 | 创建对应的 DOM 节点 |              |

### 多平台抽象能力

我们经常看到 UI = f(state) 这个公式，这个公式可以拆分为两步：
- 根据状态的变化计算出 UI；
- 根据的 UI 的变化执行具体的宿主环境 API（浏览器、小程序等）；

虚拟 DOM 只是一个对 UI 结构的描述，具体的宿主环境可以根据这个结构渲染不同的代码。

## React 中的虚拟 DOM

React 通过 JSX 来描述 UI，JSX 会被转译为一个叫做 `createElement()` 方法的调用，调用后就会得到虚拟 DOM 对象。

Babel 的编译结果如下：
![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/Pasted%20image%2020260907100425.png)
在源码中，`createElement()` 方法如下：

```js
/**
 *
 * @param {*} type 元素类型 h1
 * @param {*} config 属性对象 {id : "aa"}
 * @param {*} children 子元素 hello
 * @returns
 * <h1 id="aa">hello</h1>
 */
export function createElement(type, config, children) {
  let propName;

  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  // 说明有属性
  if (config != null) {
    // ...
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }
  
  // 经历了上面的 if 之后，所有的属性都放到了 props 对象上面
  // props ==> {id : "aa"}

  // children 可以有多个参数，这些参数被转移到新分配的 props 对象上
  // 如果是多个子元素，对应的是一个数组
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    // ...
    props.children = childArray;
  }

  // 添加默认的 props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  
  // ...
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props
  );
}

const ReactElement = function (type, key, ref, self, source, owner, props) {
    // 该对象就是最终向外部返回的 vdom（也就是用来描述 DOM 层次结构的 JS 对象）
  const element = {
    // 让我们能够唯一地将其标识为 React 元素
    $$typeof: REACT_ELEMENT_TYPE,

    // 元素的内置属性
    type: type,
    key: key,
    ref: ref,
    props: props,

    // 记录负责创建此元素的组件。
    _owner: owner,
  };
  // ...
  return element;
};
```

上面的代码中，最终返回的 element 对象就是我们所说的虚拟 DOM 对象。==官方更倾向于把这个对象称为 React 元素。==