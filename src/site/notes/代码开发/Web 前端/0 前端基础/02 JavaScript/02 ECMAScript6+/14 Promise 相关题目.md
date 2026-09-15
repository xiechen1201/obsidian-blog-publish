---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/02 ECMAScript6+/14 Promise 相关题目/","dg-note-properties":{}}
---

---
---
本文主要是结合事件循环、Promise 来练习一些题目：

## 🔢 题目一
```js
const pro = new Promise((resolve, reject) => {
  console.log(1);
  resolve();
  console.log(2);
});

pro.then(() => {
  console.log(3);
});

console.log(4);

// 输出结果：
// 1 2 4 3
```

解释：

- 首先执行全局代码，创建了一个 Promise 对象，Promise 回调函数会立即执行，输出结果 1；
- 然后调用`resolve()`方法将 Promise 的状态落定为 fulfilled；
- Promise 回调函数内部调用`resolve()`后并不会终止后续的执行，只是将任务的状态落定了，后续代码正常执行（后续无法再更改任务的状态），输出结果 2；
- 因为`pro`的状态已经落定，后续的`.then`的回调处理会被推入到「微任务」队列等待执行；
- 继续往下执行，输出结果 4；
- 这个时候全局的代码执行完成，开始事件循环机制，执行「微任务」队列，输出结果 3；

## 🔢 题目二
```js
const pro = new Promise((resolve, reject) => {
  console.log(1);

  setTimeout(() => {
    console.log(2);
    resolve();
    console.log(3);
  }, 0);
});

pro.then(() => console.log(4));

console.log(5);

// 输出结果：
// 1 5 2 3 4
```

解释：

- 首先执行全局代码，创建了一个 Promise 对象，Promise 回调函数会立即执行，输出结果 1；
- 接着函数内遇到一个定时器，然后主线程会把定时器交给相关的线程进行计时，至少在 0 秒后将定时器的回调推入到「宏任务」队列；
- 因为`pro`状态没有被落定，所以`.then`的回调必须要等到`pro`的状态落定后才会进入「微任务」队列；
- 继续执行输出结果 5；
- 这个时候全局的代码执行完成，开始事件循环机制。虽然「微任务」比「宏任务」的优先级更高，但是目前只有「宏任务」，所以执行定时器的回调，输出结果 2；
- 接着执行，将`pro`的状态落定为 fulfilled 的，同时`pro`的`.then`回调会被推入到「微任务」队列；
- 接着执行，输出结果 3（一定是执行完定时器的回调后才能开始下一次的事件循环）；
- 继续下一次的事件循环，执行「微任务」队列的`.then`回调，输出结果 4；

## 🔢 题目三
```js
const pro1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve();
  }, 1000);
});

const pro2 = pro1.catch(() => {
  return 2;
});

console.log('promise1', pro1);
console.log('promise2', pro2);

setTimeout(() => {
  console.log('promise1', pro1);
  console.log('promise2', pro2);
}, 2000);

// 输出结果：
/*
  promise1 Promise{<pending>}
  promise2 Promise{<pending>}
  promise1 Promise{<fulfilled>}
  promise2 Promise{<fulfilled>}
*/
```

解释：

- 首先执行全局代码，创建了一个 Promise 对象，Promise 回调函数会立即执行，内部创建了一个定时器，至少在 1 秒后定时器的回调会被推入到「宏任务」队列，但是目前`pro1`的状态没有落定；
- 继续往下执行遇到`pro2`，`pro2`是依赖于`pro1`的，由于`pro1`的状态没有落定，则`pro2`的状态也是落定；
- 继续执行，打印`pro1`和`pro2`的结果：`promise1 Promise{<pending>}`、`promise2 Promise{<pending>}`；
- 继续执行又遇到一个定时器，该定时器会在 2 秒后将回调推入到「宏任务」队列；
- 全局代码执行完成，开始事件循环机制。由于第一个定时器先推入「宏任务」自然先执行第一个定时器的回调，该回调内将`pro`的状态更改为 fulfilled 的。由于`pro2`是依赖于`pro1`的且`pro2`只处理了`pro1`错误的情况没有处理成功的情况，所以`pro2`的状态和`pro1`一样的；
- 执行完第一个定时器回调后，继续执行第二个定时器回调，输出结果：`promise1 Promise{<fulfilled>}`、`promise2 Promise{<fulfilled>}`;

## 🔢 题目四
```js
async function m() {
  console.log(0);
  const n = await 1;
  console.log(n);
}
m();
console.log(2);

// 输出结果：
// 0 2 1
```

解释：

- 首先执行全局代码，调用`m()`函数，输出结果 0；
- 往下执行遇到`await`修饰符，该修饰符会将 1 包装为`Promise.resolve(1)`的 Promise 对象，其状态为 fulfilled，值为 1。`await`后面的代码推入到「微任务」队列并直接结束`m()`函数的执行；
- 继续执行全局代码输出结果 2；
- 全局代码执行完成后，开始事件循环，恢复`await`后续代码的执行，输出结果 1；

`async`和`await`本质上就是一个语法糖，上面的代码实际的写法是：

```js
function m() {
  console.log(0);
  // const n = await 1;
  return new Promise((resolve, reject) => {
    resolve(1)
  }).then(n => console.log(n))
}
m();
console.log(2);
```

可以把`await`后续的代码理解为一个`.then`回调！

<br/>

## 🔢 题目五
```js
async function m() {
  console.log(0);
  const n = await 1;
  console.log(n);
}

(async () => {
  await m();
  console.log(2);
})();

console.log(3);

// 输出结果：
// 0 3 1 2
```

解释：

- 首先执行全局的代码，运行遇到立即执行函数，该函数是一个异步函数，函数内又调用了`await m()`，进入`m()`函数；
- `m()`函数也是一个异步函数，输出结果 0。继续执行遇到`await 1`，1 会被包装为一个被理解解决的 Promise 对象，状态为 fulfilled，值为 1。`await`后续的代码会被推入「微任务」队列，`m()`函数执行完成；
- 回到立即执行函数内部，后续一定要等到`m()`函数执行完成才能继续执行，`await m()`后续的代码也会被推入「微任务」队列；
- 继续执行全局代码，输出结果 3；
- 全局代码执行完成，开始事件循环机制。先执行第一个「微任务」，输出结果 1，`m()`函数所有代码执行完成；
- 接着执行第二个「微任务」，输出结果 2；

以上代码可以改写为：

```js
async function m() {
  console.log(0);
  return new Promise((resolve, reject) => {
    resolve(1);
  }).then((res) => console.log(res));
}

(() => {
  m().then(() => console.log(2));
})();

console.log(3);
```

## 🔢 题目六
```js
async function m1() {
  return 1;
}

async function m2() {
  const n = await m1();
  console.log(n);
  return 2;
}

async function m3() {
  const n = m2();
  console.log(n);
  return 3;
}

m3().then((n) => console.log(n));

m3();

console.log(4);

// 输出结果：
/*
  Promise{<pending>}
  Promise{<pending>}
  4
  1
  3
  1
*/
```

解释：

- 首先执行全局的代码，执行`m3()`函数，`m3()`函数是一个异步函数，内部又调用了`m2()`函数；
- 进入`m2()`函数内部，`m2()`函数是一个异步函数，然后又`await m1()`调用`m1()`函数；
- 进入`m1()`函数内部，`m1()`函数是一个异步函数，然后返回数字 1，所以`m1()`最终是一个状态为 fulfilled 值为 1 的 Promise 对象，`m1()`函数执行完成；
- 回到`m2()`函数，`await m1()`后续代码推入「微任务」队列，`m2()`函数执行完成，`m2()`函数最终是一个状态为 pending 的 Promise 函数；
- 回到`m3()`函数，由于调用`m2()`函数没有使用`await`，所以后续代码正常执行，输出结果`Promise{<pending>}`，返回数字 3，`m3()`函数最终是一个状态为 fulfilled 值为 3 的 Promise 对象；
- `m3()`的`.then()`推入「微任务」队列；
- 继续执行，又调用了`m3()`函数，流程同上，只是没有`m3()`的`.then()`回调，至此「微任务队列」共有：

```plain
[
  {
    // m2() 函数 await 后续代码
    console.log(n);
    return 2;
  }
  {
    // m3() 函数 .then 回调
    (n) => console.log(n)
  },
  {
    // m2() 函数 await 后续代码
    console.log(n);
    return 2;
  }
]
```

- 继续执行打印数字 4；
- 全局代码执行完成，开始事件循环机制。执行第一个「微任务」，打印结果 1；
- 执行第二个「微任务」，打印结果 3；
- 执行第三个「微任务」，打印结果 1；

## 🔢 题目七
```js
Promise.resolve(1)
  .then(1)
  .then(Promise.resolve(2))
  .then(console.log);

// 打印结果：
// 1
```

解析：

- 如果`.then`的参数不是函数则会被忽略，类似于`.then(null)`。
- 然后继续向后处理，直到`.then(console.log)`；

## 🔢 题目八
```js
var a;
var b = new Promise((resolve, reject) => {
  console.log('promise1');
  setTimeout(() => {
    resolve();
  }, 1000);
})
  .then(() => console.log('promise2'))
  .then(() => console.log('promise3'))
  .then(() => console.log('promise4'));

a = new Promise(async (resolve, reject) => {
  console.log(a);
  await b;
  console.log(a);
  console.log('after1');
  await a;
  resolve(true);
  console.log('after2');
});

console.log('end');

// 打印结果：
/*
  promise1
  undefined
  end
  promise2
  promise3
  promise4
  Promise{<pending>}
  after1
*/
```

解释：

- 开始执行全局代码，声明了变量`a`值为`undefined`；
- 声明了变量`b`值为一个 Promise 对象，实例化 Promise 对象的时候回调函数立即执行，输出结果`promise1`;
- 继续执行遇到定时器，主线程通知计时器线程开始计时，至少 1 秒后计时器线程将定时器的回调推入到「宏任务」队列中，回调函数执行完成；
- 继续执行`a`被赋值为一个 Promise 对象，实例化 Promise 对象的时候回调函数立即执行，打印`a`的结果，由于赋值过程并没有执行完成所以`a`的值依然是`undefined`（必须要等到`=`右面全部执行完成才能完成赋值）;
- 继续执行`await b`，由于`b`任务对象没有执行完成，所以后续代码暂停执行，将变量`a`赋值为`Promise{<pending>}`；
- 继续执行全局代码，输出结果`end`;
- 全局代码支持完成后开始事件循环机制，从「宏任务」队列中执行任务，将变量`b`的任务对象状态更改为 fulfilled。
- `b`的第一个`.then()`回调被添加到「微任务」队列并执行，输出结果`promise2`，`b`的第二个`.then()`回调被添加到「微任务」队列并执行，输出结果`promise3`，`b`的第三个`.then()`回调被添加到「微任务」队列并执行，输出结果`promise3`；
- 恢复`await b`的后续执行，打印`a`的结果，输出`Promise{<padding>}`;
- 继续执行打印`after1`；
- 继续执行`await a`，等待`a`执行完成（这会导致死锁，因为 a 永远不会完成）；

## 🔢 题目九
```js
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(function () {
  console.log('setTimeout');
});

async1();

new Promise(function (resolve) {
  console.log('promise1');
  resolve();
}).then(function () {
  console.log('promise2');
});

console.log('script end');

// 输出结果：
/*
  script start
  async1 start
  async2
  promise1
  script end
  async1 end
  promise2
  setTimeout
*/
```

解释：

- 首先执行全局代码，开始是函数的定义并没有调用；
- 打印结果`script start`；
- 继续执行创建了一个定时器，至少 0 秒后定时器的回调会被推入到「宏任务」队列；
- 调用`async1()`函数，打印结果`async1 start`，然后执行`await async2()`执行`async2()`函数；
- 打印结果`async2`，`async2()`函数执行完成回到`async1()`函数，`await async2()`后续代码推入「微任务」队列，`async1()`函数执行完成；
- 继续执行创建了一个 Promise 对象，回调函数立即执行，打印结果`promise1`，并将 Promise 的状态设置为 fulfilled；
- Promise 对象状态落定后，`.then`回调函数推入「微任务」队列；
- 继续执行打印结果`script end`；
- 全局代码执行完成后，开始进行事件循环机制。执行第一个「微任务」，恢复`async1()`函数`await`后续的执行，打印结果`async1 end`;
- 执行第二个「微任务」，执行 Promise 对象的`.then`回调，打印结果`promise2`；
- 执行「宏任务」，打印结果`setTimeout`；
