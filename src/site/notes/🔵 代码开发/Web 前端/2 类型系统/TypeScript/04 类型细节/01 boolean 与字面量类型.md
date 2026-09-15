---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/2 类型系统/TypeScript/04 类型细节/01 boolean 与字面量类型/","dg-note-properties":{}}
---

---
---
`number`、`string`、`boolean`、`symbol`、`bigint`这些 JS 本身就支持的基础类型使用起来很简单，TS 的书写几乎感觉不到和 JS 的区别，而且支持很多种书写方式，当然中间还隐藏着一些重要的细节。

例如`boolean`举例来说：

```ts
// 🤔 let a: boolean
let a = true;
// 🤔 let b: boolean
var b = false;
// 🤔 const c: true
const c = true;

// 🤔 let d: boolean
let d: boolean = true;
// 🤔 let e: true
let e: true = true;
// 🤔 let f: false
let f: false = false;
// 🤔 let g: true
let g: true = false; // ❌不能将类型false分配给类型true
```

通过上面的代码能够发现：

- 隐式的定义`boolean`数据，TS 推导出来的类型就是`boolean`类型，后续赋值为`true/false`均可以；
- 显式定义的`boolean`数据，后续赋值为`true/false`均可以；
- 显式定义的`true/false`字面量类型，后续无法赋值为`false/true`的数据；
- 使用`const`且隐式定义的`boolean`数据，TS 会推导为字面量类型`true/false`；

也就是说，`c`、`e`、`f`和`g`的变量赋值只能赋值为对应的布尔字面量类型。如果赋值为一个不匹配的字面量类型就像产生错误：

```ts
let g: true = false; // ❌ 不能将类型false分配给类型true
```

特别需要注意的是使用`const`声明的布尔类型变量，会被推导为一个字面量类型！

```ts
// 🤔 const c: true
const c = true;
```
