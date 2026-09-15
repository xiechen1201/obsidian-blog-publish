---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/2 类型系统/TypeScript/04 类型细节/06 never 类型/","dg-note-properties":{}}
---

---
---
`never`类型表示从不，绝不的意思，我们之前在交叉类型中见到过这个类型：

```ts
type A = string & number; // never
```

我们之前讲述的`null`，`undefined`和`void`类型都是有具体意义的，也表示具体的类型，`undefined`表示尚未定义，`null`表示缺少值，甚至是`void`就表示一个空类型，就像没有返回值的函数使用`void`来作为返回值类型标注一样。

而`never`才是一个“什么都没有”的类型，它甚至不包括空的类型，严格来说`never`类型不携带任何的类型信息。

例如下面的联合类型：

```ts
// 🤔 type Foo = string | number | boolean | void | null | undefined
type Foo = string | number | boolean | undefined | null | void | never;
```

我们把常见的基础类型都放在这个联合类型中，但是 TS 推导出来的类型却没有`never`类型了，被直接无视了。

在 TS 类型系统中，`never`是整个类型系统中最底层的类型。如果说`any`，`unknown`是每个其他类型的父类型。那么`never`就是每个其他类型的子类型。这意味着，`never`类型可以赋值给其他任何类型，但是反过来，却行不通。

```ts
let neverValue: never = undefined as never;
let stringValue: string = "";

neverValue = stringValue; // ❌ 不能将类型“string”分配给类型“never”
```

在实际的工作中，我们不会显式的声明一个`never`类型，因为没有任何的意义，它主要是被类型检查所使用的。

例如利用`never`的特性与类型的控制流分析，让 Typescript 做出更合理的处理：

```ts
type Method = "GET" | "POST";

function request(url: string, method: Method) {
    if (method === "GET") {
        // 🤔 (parameter) method: "GET"
        console.log(method);
        // do somethings...
    } else if (method === "POST") {
        // 🤔 (parameter) method: "POST"
        console.log(method);
        // do somethings...
    } else {
        // 🤔 (parameter) method: never
        console.log(method);
    }
}
```

上面的代码没有什么问题，但是假如有一天`Method`类型加入了新的联合类型，例如`"PUT" | "DELETE"`，在团队开发中`request()`函数是无感知的：

```ts
type Method = "GET" | "POST";

function request(url: string, method: Method){
  // ...正常执行，不报错
}
```

但是如果我们在`else`内部声明一个`never`类型的变量，TS 就会抛出异常：

```ts
function request(url: string, method: Method) {
    if (method === "GET") {
        console.log(method); // GET
        // do somethings...
    }
    else if (method === "POST") {
        console.log(method); // POST
        // do somethings...
    } else {
        const _neverCheck: never = method; // ❌ 不能将类型“string”分配给类型“never”
        console.log(method);
    }
}
```

这样就可以把错误扼杀在摇篮里。

<br/>tips
❕信息

这种方式也叫做「穷举式检查」，积极的对不期望的情况进行错误处理，在编译时就捕获未处理的情况，而不是默默地忽略它们。

<br/>

还有些情况使用`never`类型确实是符合逻辑的，例如一个只负责抛出错误的函数：

```ts
function fn(): never {
    throw new Error("error");
}
```

在类型流的分析中，一旦一个返回值为`never`的函数被调用，则后面的代码将会被无视：

```ts
function foo(n: number) {
    if (n > 10) {
        fn();
        let name = "jack"; // 检测到无法访问的代码。ts(7027)
        console.log("hello");
    }
}
```
