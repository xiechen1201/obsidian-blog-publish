---
{"dg-publish":true,"permalink":"//web/2/type-script/07/10-infer/","dg-note-properties":{}}
---

在 TS 中使用`infer`关键字，可以在条件类型中声明泛型类型（可以推断类型）。

示例我们之前想要获取数组元素的类型：

```ts
type Flatten<T> = T extends any[] ? T[number] : T;

type T1 = Flatten<number[]>; // number
type T2 = Flatten<string[]>; // string
type T3 = Flatten<[1, 2, 3, 4]>; // 1 | 2 | 3 | 4
const arr = [{ id: 1, name: "zhangsan" }];
type T4 = Flatten<typeof arr>; // {id: number, name: string}
```

使用`infer`可以对上面的代码进行优化：

```ts
type Flatten<T> = T extends (infer U)[] ? U : T;
```

对比第一段代码中的方括号运算符`infer U`比`T[number]`更加的易读。

`infer U`中的`U`表示待推断的类型，我们完全可以将这里的`U`看着是`any`，当执行的时候 TS 推导出的类型会赋值给`U`。

例如，我们希望获取数组第一个元素的类型：

```ts
type arr1 = ['a', 'b', 'c'];
type arr2 = [3, 2, 1];

type F1 = First<arr1>; // 'a'
type F2 = First<arr2>; //  3
```

之前的方式：

```ts
type First<T> = T extends any[] ? T[0] : never;
// 或者使用下面的方式
// 获取 length 属性的类型，其实也就是数组长度，不是0的话，得到第0个位置上元素的类型
type First<T extends any[]> =  T['length'] extends 0? never : T[0];
```

现在的方式：

```ts
type First<T extends any[]> = T extends [infer F, ...infer R] ? F : never;
```

上面的代码中，使用`infer`对`U`进行推断，将第一个元素和其余的元素进行分开，最后如何条件类型成立返回第一个元素类型即可。

继续，实现交换元组两个位置上的类型：

```ts
type Swap<T extends any[]> = T extends [infer A, infer B] ? [B, A] : T;

type S1 = Swap<[1, 2]>; // 符合元组结构，首尾元素替换[2, 1]
type S2 = Swap<[1, 2, 3, 4]>; // 不符合元组结构，直接返回原数组 [1,2,3,4]
```

如何我希望无论如何数组的首位都可以进行交换，如何做呢？可以是`...`运算符：

```ts
type Swap<T extends any[]> =
    T extends [infer A, ...infer Rest, infer B]
    ? [B, ...Rest, A]
    : T;
```

函数也可以进行推断，例如写一个类型工具获取函数的返回值类型：

```ts
type GetReturnType<T> =
  T extends (...args: any[]) => infer R
  ? R
  : never;

// string
type A = GetReturnType<() => string>;
// void
type B = GetReturnType<(n: number) => void>;
// number
type C = GetReturnType<() => number>;
```

`GetReturnType<T>`实际上是官方类型工具`ReturnType<Type>`的实现方式。
