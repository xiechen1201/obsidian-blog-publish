---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/02 ECMAScript6+/13 Promise/","dg-note-properties":{}}
---

## 基础
假如现在小明要向 4 个女孩表白，并且是串行执行的，也就是先给 A 发送短信，然后等待 A 的回复，如果 A 拒绝了表白，那就继续给 B 发送短信，继续等待 B 的回复，如果 B 答应了表白，那么就不再给 C 发送短信。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721183029371-b0bfef25-21ed-4652-aeb5-6a3e4af462c9.png)

我将发送短信这一行为封装为一个方法：

```js
// 方法接收一个 name，一个成功的回调函数，一个失败的回调函数
function sendMessage(name, onFulfilled, onRejected) {
  console.log(`向 ${name} 发送了表白信息`);
  console.log(`等待${name}回复...`);

  // 一秒钟执行执行回调函数
  setTimeout(() => {
    if (Math.random() <= 0.1) {
      onFulfilled(`${name} 同意了表白！✅`);
    } else {
      onRejected(`${name} 拒绝了表白！❌`);
    }
  }, 1000);
}
```

接下来我开始给不同的人发送短信：

```js
// 给小丽发送短信
sendMessage(
  '小丽',
  (msg) => {
    console.log(msg);
  },
  (msg) => {
    console.log(msg);

    // 给小芳发送短信
    sendMessage(
      '小芳',
      (msg) => {
        console.log(msg);
      },
      (msg) => {
        console.log(msg);

        // 给小华发送短信
        sendMessage(
          '小华',
          (msg) => {
            console.log(msg);
          },
          (msg) => {
            console.log(msg);
            console.log('注定孤独终老！🤭')
          }
        );
      }
    );
  }
);
```

通过上面这段代码的写法可以很明显的看出来一个问题，那就是一层层的回调嵌套，这就形成了回调地狱。

所以 ES6 提供了 Promise 来解决回调地狱！

## Promises/A+ 规范
Promises/A+ 规范就是一种规范，定义了 Promise 的行为和接口，也就是说规定了如何实现 Promise。

<br/>tips
A+ 并没有特别的含义，只是用来表示这是一个增强或者改进的版本。A+ 规范是在原来的 Promises/A 的规范上进行拓展和改进下而来的。

Promises/A+ 规范：[https://promisesaplus.com/](https://promisesaplus.com/)

<br/>

Promises/A+ 的规定：

1、所有的异步场景，都可以看作是一个异步任务，每个异步任务在 JS 中应该表现为一个对象，该对象称为 Promise 对象，也叫任务对象。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721183534859-921a43d8-6e8d-415c-9ea6-62ab72e003b5.png)

2、每个任务对象都应该有「两个阶段」、「三种状态」。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721183575225-f8e95560-ecf8-4e50-a556-0398dc8af5b8.png)

根据常理，阶段和状态之间存在以下的逻辑：

- 当任务从「未决阶段」改变为「已决阶段」后，无法逆行；
- 当任务从「挂起状态」改变为「完成状态」或者「失败状态」后，无法逆行；
- 当任务一旦完成或者失败，那么任务的状态也就落定了，是永远无法改变的；

3、挂起==>完成，称之为`resolve`。挂起==>失败，称之为`reject`。当任务完成时，可能有一个相关数据。任务失败时，可能有一个失败原因。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721183888714-9e7dbadd-e027-4b3a-b397-80c8ade6bcc8.png)

4、可以针对任务进行后续处理，针对完成状态的后续处理称之为`onFulfilled`，针对失败的后续处理称之为`onRejected`。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721184030313-be777b61-1f6e-49d1-b5b4-c6111b8eb86e.png)

## Promise API
ES6 提供了一套 API，实现了 Promises/A+ 规范。

基本使用：

```js
// 创建一个任务对象，该任务立即进入 pending 状态
const pro = new Promise((resolve, reject) => {
  // 任务的具体执行流程，该函数会立即被执行
  // 调用 resolve(data)，可将任务变为 fulfilled 状态， data 为需要传递的相关数据
  // 调用 reject(reason)，可将任务变为 rejected 状态，reason 为需要传递的失败原因
});

pro.then(
  (data) => {
    // onFulfilled 函数，当任务完成后，会自动运行该函数，data为任务完成的相关数据
  },
  (reason) => {
    // onRejected 函数，当任务失败后，会自动运行该函数，reason为任务失败的相关原因
  }
);
```

有了 Promise API 我可以将本文开头的表白案例进行改造一下，使用 Promise 的写法书写为一个异步任务：

```js
function sendMessage(name) {
  // 返回一个任务对象
  return new Promise((resolve, reject) => {
    console.log(`向 ${name} 发送了表白信息`);
    console.log(`等待${name}回复...`);

    setTimeout(() => {
      if (Math.random() <= 0.1) {
        // 将异步任务更改为 fulfilled 状态
        resolve(`${name} 同意了表白！✅`);
      } else {
        // 将异步任务更改为 rejected 状态
        reject(`${name} 拒绝了表白！❌`);
      }
    }, 3000);
  });
}
```

然后调用任务函数：

```js
// 向小丽表白
sendMessage('小丽').then(
  (res) => console.log(res), // 针对任务完成状态的 onFulfilled 函数
  (err) => { // 针对任务失败状态的 onRejected 函数
    console.log(err);

    // 向小芳表白
    sendMessage('小芳').then(
      (res) => console.log(res),
      (err) => {
        console.log(err);

        // 向小华表白
        sendMessage('小华').then(
          (res) => console.log(res),
          (err) => {
            console.log(err);
            console.log('注定孤独终老！🤭');
          }
        );
      }
    );
  }
);
```

将改写后的案例和上面中 Promises/A+ 规范进行贴图理解：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721184527095-4e4fec7c-dda6-46df-9f87-3b831539efe1.png)

但其实回调地狱的问题仍然没有被解决，只是将回调函数的嵌套更改为了`.then`的嵌套。

换成别的案例也是同样的写法，例如我想封装一个异步函数，函数主要是帮我创建一张图片，图片加载成功后执行`resolve()`方法，失败则加载`reject()`方法：

```js
function createImage(imgUrl) {
  // 创建异步任务
  return new Promise((resolve, reject) => {
    const img = document.createElement('img');
    img.src = imgUrl;
    img.width = '500';

    img.onload = () => resolve(img); // 图片加载成功任务状态更改为 fulfilled
    img.onerror = (e) => reject(e);  // 图片加载失败任务状态更改为 rejected
  });
}

createImage('https://element.eleme.cn/static/theme-index-blue.c38b733.png')
  .then(
    // onFulfilled 函数
    (res) => {
      document.body.appendChild(res)
    },
    // onRejected 函数
    (err)=>{
      console.log(err)
    })
```

### > .catch() 方法
某些时候我只想处理失败不想处理成功的状态，那么就可以使用`.catch()`方法。

```js
new Promise((resolve, reject)=>{
  reject('错误！')
}).catch(error => console.log(error));
```

其实`.catch(onRejected)`=`.then(null, onRejected)`方法！

### > .finally() 方法
`.finally()`方法在异步任务状态变为 fulfilled 或 rejected 时都会执行。这个方法可以避免`onResolved`和 `onRejected`处理程序中出现冗余代码。

```js
let pro1 = new Promise((resolve, reject)=>{
  resolve()
});
let pro2 = new Promise((resolve, reject)=>{
  reject()
})

let onFinally = function() {
  setTimeout(console.log('Finally!'), 0);
}

pro1.finally(onFinally); // Finally
pro2.finally(onFinally); // Finally
```

## 链式调用
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721185953304-2767a957-1f6a-4eff-8c66-c630663be855.png)

在 Promise API 中，被`.then()`处理后「必定返回一个新的 Promise」，也可以理解为返回一个新的异步任务。

例如读书的时代必定会经历这样的流程：学习->考试->出成绩->读大学。

针对这个案例就可以使用链式调用来一步一步的进行处理：

```js
const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  resolve();
});

const pro2 = pro1.then((data) => {
  console.log('考试');
});

const pro3 = pro2.then((data) => {
  console.log('出成绩');
});

console.log(pro3);
```

「新异步任务」的状态取决于「前任务后续的处理」：

- 若没有相关的后续处理，新任务的状态和前任务的状态一致，数据为前任务的数据；

```js
// 只处理了成功的情况，没有处理失败的情况

const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  console.log('中奖了，不用学习了');
  // 状态落定为 rejected
  reject();
});

// pro1（前任务）没有处理失败的情况
const pro2 = pro1.then((data) => {
  console.log('考试');
});

// 导致 pro2 （新任务）的状态也跟着失败，因为后续无法进行了
setTimeout(() => console.log(pro2), 0);  // Promise{<rejected>}
```

```js
// 只处理了失败的情况，没有处理成功的情况

const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  // 状态落定为 fulfilled
  resolve();
});

// pro1 (前任务)的状态为成功，pro2 (新任务)的状态也为成功！
const pro2 = pro1.catch((data) => {
  console.log('学习失败了');
});

setTimeout(() => console.log(pro2), 0); // Promise{<fulfilled>}
```

- 若有后续处理但是还没执行，则新任务也会挂起；

```js
const pro1 = new Promise((resolve, reject) => {
  console.log('学习');

  setTimeout(() => {
    resolve();
  }, 2000);
});

// pro1 状态没有落定，pro2 的状态也不会落定
const pro2 = pro1.catch((data) => {
  console.log('学习失败了');
});

setTimeout(() => console.log(pro2), 0); // Promise{<pending>}
```

- 若后续处理执行了，则根据后续处理的情况来确定新任务的状态；
    - 后续处理执行无报错，则新任务的状态为完成，数据为后续处理的返回值；

```js
const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  // 状态更改为 fulfilled
  resolve();
});

const pro2 = pro1.then((data) => {
  // 对前任务进出完成的处理，.then() 不产生错误，则 pro2 的状态即为成功！
  console.log('考试');
  return '100分';
});

setTimeout(() => console.log(pro2), 0); // Promise{<fulfilled>: 100分}
```

```js
const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  // 状态更改为 rejected
  reject('中奖了，不用学习了');
});
const pro2 = pro1.catch((error) => {
  // 处理的过程中不报错，pro2 的状态仍热为成功
  console.log('在家养老吧');
});

setTimeout(() => console.log(pro2), 0); // Promise{<fulfilled>}
```

    - 后续处理执行有错，新任务的状态为失败，数据为异常对象;

```js
const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  // 状态更改为 fulfilled
  resolve();
});
const pro2 = pro1.then((data) => {
  console.log('考试');
  // 处理过程中如果发送错误，则 pro2 的状态即为失败
  throw new Error('考试失败');
});

console.log(pro2); // Promise {<rejected>: Error: 考试失败}
```

```js
const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  // 状态更改为 rejected
  reject('中奖了，不用学习了');
});
const pro2 = pro1.catch((error) => {
  // 处理过程中如果发送错误，则 pro2 的状态即为失败
  throw new Error('颁奖的机构跑路了');
});

console.log(pro2); // Promise {<rejected>: Error: 颁奖的机构跑路了}
```

到这里简单总结一下：如果`pro1`的状态被`.then`处理了，那么`pro2`的状态取决于后续处理过程情况（不管是成功函数还是失败函数）。

<br/>

    - 后续处理如果返回一个任务对象，新任务的状态和数据与该任务对象一致;

```js
const pro1 = new Promise((resolve, reject) => {
  console.log('学习');
  // 状态更改为 rejected
  resolve();
});
const pro2 = pro1.then((data) => {
  // 返回一个 Promise 任务对象
  // pro2 的状态和 new Promise() 的状态一致！
  return new Promise((resolve, reject) => {
    // 本 Promise 和 pro2 的状态都为 rejected
    reject();
  });
});

console.log(pro2); // Promise {<rejected>}
```

由于链式任务的存在，异步代码拥有了更强的表达能力：

```js
// 常见任务处理代码

/*
 * 任务成功后，执行处理1，失败则执行处理2
 */
pro.then(处理1).catch(处理2)

/*
 * 任务成功后，依次执行处理1、处理2
 */
pro.then(处理1).then(处理2)

/*
 * 任务成功后，依次执行处理1、处理2，若任务失败或前面的处理有错，执行处理3
 */
pro.then(处理1).then(处理2).catch(处理3)
```

现在我仍然可以使用链式调用的方式改造一下小明表白的案例。

`sendMessage()`方法不变：

```js
function sendMessage(name) {
  // 返回一个任务对象
  return new Promise((resolve, reject) => {
    console.log(`向 ${name} 发送了表白信息`);
    console.log(`等待${name}回复...`);

    setTimeout(() => {
      if (Math.random() <= 0.1) {
        // 将异步任务更改为 fulfilled 状态
        resolve(`${name} 同意了表白！✅`);
      } else {
        // 将异步任务更改为 rejected 状态
        reject(`${name} 拒绝了表白！❌`);
      }
    }, 3000);
  });
}
```

主要是使用链式调用的方式：

```js
sendMessage('小丽')
  .catch((error) => {
    console.log(error);
    return sendMessage('小芳');
  })
  .catch((error) => {
    console.log(error);
    return sendMessage('小华');
  })
  .then(
    (data) => {
      console.log(data);
      console.log('小明终于找到了自己的伴侣');
    },
    (error) => {
      console.log(error);
      console.log('注定孤独终老！🤭');
    }
  );
```

以上代码，如果上一个任务状态为 rejected 那么就会被`.catch()`方法处理，如果有一个任务的状态为 fulfilled 则会直接执行到`.then(onFulfilled)`方法内。

### > 练习题
```js
const pro1 = new Promise((resolve, reject) => {
  setTimeout(() => resolve(100), 1000);
});

const pro2 = pro1.then((data) => {
  console.log(data);
  return data + 10;
});

const pro3 = pro2.then((data) => {
  console.log(data);
});

console.log(pro1, pro2, pro3); // Promise{<pending>} Promise{<pending>} Promise{<pending>}

setTimeout(() => {
  console.log(pro1, pro2, pro3); // Promise{<fulfilled>: 100} Promise{<fulfilled>: 110} Promise{<fulfilled>}
}, 2000);
```

以上代码中，因为`pro1`的状态被挂起，而`pro2`的状态是依赖`pro1`的，`pro3`的状态是依赖`pro2`的，所以刚开始都是 pending 状态。

1 秒钟后，`pro1`的状态被更改为 fulfilled 的，`pro1.then()`进行了处理且没有报错，所以`pro2`的状态也为 fulfilled，`pro2.then()`也进行了处理且没有报错，所以`pro3`的状态为 fulfilled。

```js
new Promise((resolve, reject) => {
  setTimeout(() => resolve(1), 1000);
})
  .then((data) => {
    console.log(data); // 1
    return 2;
  })
  .catch((error) => {
    throw 3;
  })
  .then((data) => console.log(data)); // 2
```

上面的代码会输出 1、2。

因为最初的任务对象在 1 秒后状态更改为 fulfilled，后续也进行了处理所以打印了 1，然后返回数字 2。因为`.then()`返回一个新的 Promise，又因为新任务的状态为 fulfilled 且没有进行相应的处理，只能继续往后执行到`.then()`方法，然后打印了 2。

把这面的题目进行改造一下：

```js
const pro = new Promise((resolve, reject) => {
  setTimeout(() => resolve(1), 1000);
})
  .then((data) => {
    console.log(data);
    return 2;
  })
  .catch((error) => {
    throw 3;
  })
  .then((data) => console.log(data));

setTimeout(() => {
  console.log(pro); // Promise{<fulfilled>}
}, 2000);
```

这里的`pro`打印的结果是`Promise{<fulfilled>}`，这是因为`pro`的状态取决于最后一个任务的处理状态！和`xxx.aaa().bbb().ccc().ddd()`是一个道理。

```js
// fulfilled
new Promise((resolve, reject) => {
  resolve();
})
  // rejected
  .then((data) => {
    data.toString();
    return 2;
  })
  // fulfilled
  .catch((err) => {
    return 3;
  })
  // fulfilled
  .then((res) => {
    console.log(res); // 3
  });
```

以上代码能够打印出 3 是因为`.then()`方法内的`data.toString()`发生了报错（因为`undefined`无法调用`.toString()`方法），然后被后续的`.catch()`处理了，`.catch()`内部没有错误所以继续往下执行到`.then()`方法，打印出数字 3。

## Promise 静态方法
### > Promise.resolve()
该方法直接返回一个完成状态的任务。

```js
const pro = Promise.resolve(100);
console.log(pro); // Promise{<fulfilled> 100}

// 等价于
new Promise((resolve, reject)=> resolve(100));
```

### > Promise.reject()
该方法直接返回一个拒绝状态的任务。

```js
const pro = Promise.reject('失败');
console.log(pro); // Promise{<rejected> '失败'}

// 等价于
new Promise((resolve, reject)=> reject('失败'));
```

### > Promise.all([])
该方法接收一个异步任务数组，返回一个异步任务。

如果数组中的异步任务「全部成功」，则该任务的状态即为成功，值为每个成功任务返回的值；如果数组中有一个异步任务失败，则该任务的状态即为失败，值为第一个失败任务的原因。

```js
const pro = Promise.all([
  Promise.resolve('成功'),
  Promise.resolve('成功'),
  Promise.resolve('成功')
]);
setTimeout(() => console.log(pro), 0); // Promise {<fulfilled> [ '成功', '成功', '成功' ]}
```

```js
const pro = Promise.all([
  Promise.resolve('成功'),
  Promise.reject('失败'),
  Promise.resolve('成功')
])

setTimeout(() => console.log(pro), 0); // Promise {<rejected>: '失败'}
```

### > Promise.any([])
该方法接收一个异步任务数组，返回一个异步任务。

该方法和`Promise.any([])`是相反的，它更强调的是「任意」，如果数组中有一个任务的状态为成功，则整个任务即为成功，值为第一个成功任务返回的值；如果数组中每个任务都失败了，则整个任务即为失败，值为一个错误对象，通过`.errors`属性可以得到每个失败任务的原因组成的数组。

```js
const pro = Promise.any([
  Promise.resolve('成功'),
  Promise.reject('失败'),
  Promise.resolve('成功')
]);
setTimeout(() => console.log(pro), 0); // Promise {<fulfilled> '成功' }
```

```js
const pro = Promise.any([
  Promise.reject('失败'),
  Promise.reject('失败'),
  Promise.reject('失败')
]);

pro.catch(error=>{
  console.log(error.errors) // // 返回一个错误对象.errors
})

setTimeout(() => console.log(pro), 0); // Promise {<rejected> 'AggregateError: All promises were rejected' }
```

### > Promise.allSettled([])
该方法接收一个异步任务数组，返回一个异步任务。

当数组中的每个方法状态都落定后（无论落定为成功或者失败），则整个任务的状态即为成功，该方法不会返回失败的任务，值为`{ status: "fulfilled", value: xx }`或者`{ status: "rejected", reason: xx }`的对象数组：

```js
const pro = Promise.allSettled([
  Promise.reject('失败'),
  Promise.resolve('成功'),
  Promise.resolve('成功')
]);
setTimeout(() => console.log(pro), 0);

/*
  Promise {
    <fulfilled>
    [
      { status: 'rejected', reason: '失败' },
      { status: 'fulfilled', value: '成功' },
      { status: 'fulfilled', value: '成功' }
    ]
  }
*/
```

### > Promise.race([])
该方法接收一个异步任务数组，返回一个异步任务。

当数组中任意一个任务的状态落定（无论落定为成功还是失败），则整个任务的状态和第一个落定状态的任务保持一致！

```js
const pro = Promise.race([
  Promise.reject('失败'),
  Promise.resolve('成功')
]);
setTimeout(() => console.log(pro), 0); // Promise {<rejected>: '失败'}
```

```js
const pro = Promise.race([
  Promise.resolve('成功'),
  Promise.reject('失败')
]);
setTimeout(() => console.log(pro), 0); // Promise {<fulfilled>: '成功'}
```

### > 练习题
```js
// 模拟一个获取学生列表的接口，返回一个异步任务
function fetchStudents(page) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() < 0.2) {
        reject(new Error('请求失败，获取第' + page + '页数据失败！'));
        return;
      }

      const stus = new Array(10).fill(null).map((el, index) => ({
        id: `NO.${(page - 1) * 10 + index + 1}`,
        name: `学生${(page - 1) * 10 + index + 1}`
      }));
      resolve(stus);
    }, Math.floor(Math.random() * 5000));
  });
}
```

1、请求接口，当任意一个任务失败，则整个任务不再继续。

```js
Promise.all([
  fetchStudents(1),
  fetchStudents(2),
  fetchStudents(3),
  fetchStudents(4),
  fetchStudents(5)
]).then(
  (results) => {
    console.log(results.flat(Infinity));
  },
  (err) => {
    console.log(err);
  }
);
```

2、请求接口，如果某页的数据存在异常，就不加入该数据。

```js
Promise.allSettled([
  fetchStudents(1),
  fetchStudents(2),
  fetchStudents(3),
  fetchStudents(4),
  fetchStudents(5)
]).then((results) => {
  // 只要状态为 fulfilled 的项
  let arr = results.filter((el) => el.status === 'fulfilled').map((el) => el.value).flat();
  console.log(arr)
});
```

3、请求接口，打印最先获取到结果的数据，如果全部失败，则打印所有的错误信息。

```js
Promise.any([
  fetchStudents(1),
  fetchStudents(2),
  fetchStudents(3),
  fetchStudents(4),
  fetchStudents(5)
])
  .then((res) => {
    console.log(res);
  })
  .catch((err) => {
    console.log(err.errors);
  });
```

4、请求接口，输出最先得到的结果的项。

```js
Promise.race([
  fetchStudents(1),
  fetchStudents(2),
  fetchStudents(3),
  fetchStudents(4),
  fetchStudents(5)
]).then(
  (res) => {
    console.log(res);
  },
  (err) => {
    console.log(err);
  }
);
```
