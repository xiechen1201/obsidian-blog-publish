---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/2 类型系统/TypeScript/04 类型细节/19 控制流分析和 typeof/","dg-note-properties":{}}
---

## 控制流分析
Typescript 具有非常强大的类型推导能力，不单单具有之前我们提到的类型拓宽，还可以进行类型的收缩。

例如在类型拓宽中，我们提到的使用`const`声明的变量会自动转变为类型的字面量。

```ts
// 🤔 let str: string
let str = "Hello";

// const 声明的变量类型就是 World 的字面量类型。
// 🤔 const str1: "World"
const str1 = "World";
```

Typescript 还可以根据我们的代码逻辑，不断的尝试类型收缩，这种能力被称为「类型的控制流分析」（也可以简单的理解为就是类型推导）。

```ts
type MyTypes = number | string | boolean | null | undefined;

// value 本来是一个联合类型
function parse(value: MyTypes) {
    // 当 value 进行 if 判断的时候，TS 会根据判断条件推断出 value 的类型
    // 每经过一个 if 判断，value 的类型就会缩小一次
    if (typeof value === "string") {
        // 🤔 (parameter) value: string
        return value;
    } else if (typeof value === "number") {
        // 🤔 (parameter) value: number
        return value * 2;
    } else if (typeof value === "boolean") {
        // 🤔 (parameter) value: boolean
        return value;
    } else {
        // 🤔 value: null | undefined
        return value;
    }
}
```

## typeof 类型查询
在前面的代码段中我们使用了`typeof`来判断`value`的数据类型，在 JavaScript 这是非常常见的用法，并且会返回`"string"`/`"number"`/`"boolean"`/`"function"`/`"object"`等值。

在 TypeScript 中给`typeof`操作符还赋予了新的功能：类型查询（Type Query Operator）。简单来说。就是通过`typeof`来获取自动推导出的类型，给`typeof`一个值，就可以帮你推导出这个值的类型。

```ts
let temp1 = "hello1";
const temp2 = "hello2";
const temp3 = null;
const temp4 = (a: string) => a.toUpperCase();

type Temp1 = typeof temp1; // string
type Temp2 = typeof temp2; // hello2
type Temp3 = typeof temp3; // null
type Temp4 = typeof temp4; // (a: string) => string
```

对象也可以使用`typeof`进行获取：

```ts
const user = {
    name: "jack",
    age: 18,
    address: {
        province: "四川",
        city: "成都"
    }
};

// 得到整个 user 对象的字面量类型
// 🤔 type User = { nage: string; age: number; address: { city: string; street: string; }; }
type User = typeof user;
```
