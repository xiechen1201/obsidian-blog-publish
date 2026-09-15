---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/2 类型系统/TypeScript/04 类型细节/03 symbol 类型/","dg-note-properties":{}}
---

---
---
`symbol`类型是 ES6 新增的一种基本数据类型。

`symbol`经常用于代替对象和映射的字符串键，确保使用正确的键，以防键被意外的设置。

```ts
let a = Symbol('a');
let b: symbol = Symbol('a');

console.log(a === b); // false

let obj = {
  name: 'Symbol',
  [a]: 'jack',
  [b]: function () {
    console.log('ts');
  }
};
console.log(obj); // {name: 'Symbol', Symbol(a): 'jack', Symbol(a): ƒ}

for (let key in obj) {
  // symbol 属性是不能被枚举的
  console.log('---', key); // --- name
}
```

定义一个`symbol`类型的数据：

```ts
// 🤔 let c: symbol
let c = Symbol('a'); // typeof c
c = 123; // ❌ 不能将类型“number”分配给类型“symbol”
```

如果使用`const`声明的`symbol`将会是`unique symbol`类型：

```ts
const c = Symbol('a'); // typeof c
```

当显式的定义`unique symbol`的时候必须使用`const`关键字：

```ts
const d: unique symbol = Symbol('a'); // typeof d
let e: unique symbol = Symbol('a'); // ❌ unique symbol的变量必须为const
```

不要将两个`symbol`类型的数据进行比较，因为是没有意义的：

```ts
console.log(c === c);
console.log(c === d); // ❌ 此比较没有意义，类型typeof c和typeof d没有重叠
```

`unique symbol`类型与其他字面量类型其实是一样的，比如`1`、`true`、`"hello"`，创建的是表示特定符号的类型。
