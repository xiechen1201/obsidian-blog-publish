---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/02 ECMAScript6+/16 async 和 await/","dg-note-properties":{}}
---

到目前为止，我们可以发现`Promise`并没有消除回调，但是消除了回调地狱，改成使用`.then`进行链式操作让程序可以有序的进行。

ES2016 推出了两个关键字`async`和`await`，用于简化`Promise`的使用。

## async
使用`async`关键字是用来修饰函数的，必须书写在函数的最前面。

```js
async () => {}; // 箭头函数
async function(){} // 匿名函数
(async function(){})() // 立即执行函数
```

被`async`修饰的函数，一定返回的是`Promise`异步任务（且不论任务的状态）！

```js
function m(){}
console.log(m()); // undefined

async function m1(){}
console.log(m1()); // Promise {<fulfilled>: undefined}
```

`async`本身就是一个语法糖，本质上没有功能性的变化，只是相比之前写起来更加的舒服！

<br/>

```js
// 等价于
function m1(){
  return new Promise((resolve, reject)=> resolve());
}
console.log(m1());
```

被`async`修饰的函数返回值即为异步任务的结果：

```js
async function m1() {
  return 'hello';
}
console.log(m1()); // Promise {<fulfilled>: 'hello'}

// 等价于
function m1() {
  return new Promise((resolve, reject) => resolve('hello'));
}
```

如果函数内返回一个`Promise`，那么这个`Promise`的状态会决定`async`函数的状态：

```js
async function foo() {
  return Promise.reject('错误！');
}
console.log(foo()); // Promise {<rejected>: '错误！'}
```

如果`async`函数内发生报错，则返回的`Promise`的状态为`rejected`：

```js
async function foo() {
  throw new Error('错误！');
}
console.log(foo()); // Promise {<rejected>: Error: 错误！}
```

## await
`await`表示「等待」某个`Promise`完成，它必须书写在`async`函数内部！

当`Promise`状态落定为成功后，可以得到`Promise`的结果。

只有`await`的`Promise`的状态落定后，才会继续执行后续的代码（仅`async`函数内）。

```js
function getList() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('数据');
    }, 1000);
  });
}

async function foo() {
  let result = await getList();
  console.log(result);

  // 类似于 getList().then(res=> console.log(res));
  // await 方便了我们的写法
}
foo();
```

`await`也可以等待其他的数据：

```js
async function foo() {
  // 如果等待的不是 Promise，会转换为 Promise
  // 等于 await Promise.resolve(100);
  const n = await 100;
  console.log(n);
}
foo();
```

如果`await`的任务失败了，需要使用`try...catch...`来进行捕获!

```js
async function foo() {
  try {
    await Promise.reject('错误！');
  } catch (error) {
    console.log(error);
  }
}

foo();
```
