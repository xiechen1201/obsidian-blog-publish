---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/12-worker/","dg-note-properties":{}}
---

前端开发中常说 JS 是单线程的语言，单线程就意味着不能像其他多线程语言那样把工作委托给独立的线程进行工作。
假如 JS 是一个多线程的语言，那么 DOM 的操作就会出现问题。

为什么 JS 单线程操作 DOM 不会存在问题？
```md file:index.md
任务1
 ↓
任务2
 ↓
任务3
```
此时因为 JS 是单线程的，只有一个线程可以用来操作 DOM，所以 DOM 的执行是串行的。

如果 JS 有 A、B、C 三个线程，三个线程同时操作 DOM，那么就会出现：
- 状态混乱
- 覆盖
- 顺序错乱
的一些问题。

正是因为这些问题，也凸显出了「工作者线程」的作用。
> [!info]
> 允许主线程把一些工作交接给其他的线程，而不更改现有的单线程模型。

JS 的工作环境是运行在操作系统中的一个虚拟环境，浏览器每打开一个新的页面，就会分配一个新的环境，每个页面都有自己的内存、事件循环、DOM 等等（相当于一个沙盒，不会影响其他的页面）。
使用工作者线程可以让页面多一个完全独立的子线程，这个线程不能操作 DOM，但是可以和主线程并行执行代码。

## 🔢 工作者线程和线程（对比）

共同之处：
- 工作者线程就是实际的线程实现的；
	- 也就是说并不是 JS 模拟的线程，而是浏览器真正创建了一个子线程；
- 工作者线程和页面并行执行；
- 工作者线程可以共享某些内存；
- 工作者线程和页面可能不在同一个进程内（主要看浏览器引擎的实现）；
- 创建工作者线程具有一定的代价；
	- 因为有独立的事件循环、全局对象、事件处理，所以也会存在一些代价；

> [!info]
> HTML Web Worker 规范是这样说的：
> 工作者线程相对比较重，不建议大量使用。例如，对一张 400 万像素的图片，为每个像素 都启动一个工作者线程是不合适的。通常，工作者线程应该是长期运行的，启动成本比较高， 每个实例占用的内存也比较大。

<br />

## 🔢 工作者线程的类型

工作者线程定义了三种主要的线程：
- 专用工作者线程
	- 有时被称为 Web Worker，可以让脚本单独创建一个 JS 线程，来执行委托的任务，只能被创建它的页面所使用；
- 共享工作者线程
	- 和专用线程类似，主要区别是可以被多个不同的页面使用，前提是和创建共享线程的页面是同源的；
- 服务工作者线程
	- 和上面两种线程完全不同，主要是用于拦截、重定向和修改页面发出的请求；

## 🔢 专用工作者线程

这个线程是 Web 最常用的线程，也是最简单的线程，可以和主线程进行交换信息、发送网络请求
执行密集型计算、处理大量数据等等不适合在页面执行的任务（导致页面卡顿的任务）。

1、创建专用工作者线程
最简单的方式就是通过`new Worker()`然后传入一个文件的地址，然后就会在后台异步加载脚本并实例化工作者线程。
```js file:index.js
const worker = new Worker('./worker.js');
console.log("🚀 ~ worker:", worker)
```

2、工作者线程的安全限制
线程脚本只能从和父页面同源的地方加载，非同源会加载失败（受同源策略的限制）。
```js file:index.js
// 尝试基于 https://example.com/worker.js 创建工作者线程
const sameOriginWorker = new Worker('./worker.js');

// 尝试基于 https://wiley.com/worker.js 创建工作者线程
const remoteOriginWorker = new Worker('https://wiley.com/worker.js');

// Error: Uncaught DOMException: Failed to construct 'Worker':
// Script at https://wiley.com/main.js cannot be accessed
// from origin https://example.com
```

3、使用 Worker 对象
Worker 对象是和专用工作者线程的连接点，它可以用于在两个线程之间传递信息，捕获线程之间的事件。

Worker 对象的事件：
- `onerror`，该事件会在工作者线程发送错误的时候触发；
- `onmessage`，该事件会在工作者线程向父上下文发送信息的时候触发；
- `onmessageerror`，该事件在工作者线程收到无法反序列化的信息时触发；

Worker 对象的方法：
- `postMessage()`，通过异步消息给工作者线程发送信息；
- `terminate()`，用于立即终止工作者线程，不会等待工作者线程的代码全部执行完；

> [!info]
> 工作者线程内部有一个`self`全局对象（类似于`window`对象），是`WorkerGlobalScope`对象的实例。
> `self`对象上可用的属性是`window`对象上严格的子集，某些属性会返回特定于工作者线程的版本。
> 类似的，`self`对象的方法也是`window`对象方法的子集，这些方法和`window`对象的方法操作一致。

4、`DedicatedWorkerGlobalScope`
在专用工作者线程内部，全局的作用域是`DedicatedWorkerGlobalScope`，其继承于`WorkerGlobalScope`对象，工作者线程可以通过`self`对象来访问全局的作用域。
```js file:worker.js
console.log("🚀 ~ worker.js ~ self:", self)
```
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/image.png)

`DedicatedWorkerGlobalScope`在`WorkerGlobalScope`对象的基础上又新增了以下的属性和方法：
- `name`，提供给`Worker`构造函数的一个可选的字符串标识符；
- `postMessage()`，和`worker.postMessage()`对应的方法，用于从工作者线程内部向父线程发送信息；
- `close()`，用于通知工作者线程关闭，丢弃事件循环中已经排队的任务，并阻止继续添加新的任务；
- `importScripts()`，用于向工作者线程中导入任意数量的脚本；

## 🔢 专用工作者线程的生命周期

当脚本中调用`new Worker()`的时候，会初始化对工作者线程的创建，然后把`Worker`返回给父上下文，但是关联的工作者线程可能还没有创建好。

初始化的时候，虽然子线程尚未执行，但是可以把任务信息先加入队列中，等到工作者线程的状态可用时，再把这些消息添加到消息队列。
```js file:index.js
const worker = new Worker('./worker.js');

worker.postMessage("foo");
worker.postMessage("bar");
worker.postMessage("baz");
```

```js file:worker.js
self.onmessage = (e) => {
  console.log("🚀 ~ worker.js ~ e:", e)
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/image-1.png)

当工作者线程创建后，那么它就会伴随着整个生命周期而存在，触发调用`self.close()`或者`worker.terminate()`。即使工作线程运行完成，线程的环境也依旧存在，不会被垃圾机制进行回收。

<br />

自我终止和外部终止最终都会执行相同的工作者线程终止例程，例如：
```js file:index.js
const worker = new Worker('./worker.js');
worker.onmessage = (e) => {
  console.log("🚀 ~ worker.js ~ e:", e)
}

// 🚀 ~ worker.js ~ e: MessageEvent {isTrusted: true, data: 'foo', origin: '', lastEventId: '', source: null, …}
// 🚀 ~ worker.js ~ e: MessageEvent {isTrusted: true, data: 'bar', origin: '', lastEventId: '', source: null, …}
```

```js file:worker.js hl:2
self.postMessage("foo");
self.close()
self.postMessage("bar");
setTimeout(() => {
  self.postMessage("baz");
}, 0)
```

虽然工作者线程中调用了`close()`方法，但显然工作者线程并没有立即终止。==`close()`只是通知工作者线程取消事件循环中的所有任务，并阻止添加新的任务，这就是为什么"baz"没有被打印的原因。==

如果是调用`terminate()`方法则会立即进行终止：
```js file:index.js
const worker = new Worker('./worker.js');
setTimeout(() => {
  worker.postMessage("foo");
  worker.terminate();
  worker.postMessage("bar");
  setTimeout(() => {
	worker.postMessage("baz");
  }, 0)
}, 1000)
```

```js file:worker.js
self.onmessage = (e) => {
  console.log("🚀 ~ worker.js ~ e:", e)
}

// 🚀 ~ worker.js ~ e: MessageEvent {isTrusted: true, data: 'foo', origin: '', lastEventId: '', source: null, …}
```

可以看到，当调用了`terminate()`方法后，工作者线程的消息队列就会被清理并锁住，所以只打印了一个"foo"。

在整个生命周期中，一个专用的工作者线程只会关联一个网页，除非手动终止，否则只要网页存在，那么工作者线程就会存在。如果页面进行了导航，则会将其关联的工作者线程标记为终止。

## 🔢 配置 Worker 选项

`Worker()`函数还可以将一个配置对象作为第二个参数，该对象支持下列参数：
- `name`，在工作者线程中通过`self.name`获取到的字符串标识；
- `type`，表示脚本的运行方式，可以是`"classic"`或者`"module"`；
- `credentials`，如果`type`为`module`，指定浏览器在请求`worker`模块文件时，要不要携带 cookie、认证信息等凭证；
	- 值可以为`omit`、`same-origin`、`include`，这些值和`fetch`相同；

## 🔢 在 JavaScript 行内创建工作者线程

工作者线程需要基于脚本文件进行创建，但并不意味着脚本必须是远程资源，可以使用`Blob`对象 URL 在行内脚本创建。这样可以更快的初始化工作者线程，因为不需要进行网络加载。

```js file:index.js
const workerScript = `
	self.onmessage = (e) => {
	  console.log("🚀 ~ worker.js ~ e:", e)
	}
`;

// 创建 Blob 对象
const workerScriptBlob = new Blob([workerScript], { type: 'text/javascript' });
const workerScriptURL = URL.createObjectURL(workerScriptBlob);

// 初始化 Worker
const worker = new Worker(workerScriptURL);
worker.postMessage("foo");
worker.postMessage("bar");
worker.postMessage("baz");

// 🚀 ~ worker.js ~ e: MessageEvent {isTrusted: true, data: 'foo', origin: '', lastEventId: '', source: null, …}
// 🚀 ~ worker.js ~ e: MessageEvent {isTrusted: true, data: 'bar', origin: '', lastEventId: '', source: null, …}
// 🚀 ~ worker.js ~ e: MessageEvent {isTrusted: true, data: 'baz', origin: '', lastEventId: '', source: null, …}
```

在这个案例中，通过字符串创建了`Blob`，然后又通过`Blob`创建了对象 URL，最后又传给了`Worker()`构造函数，这样的方式同样创建了工作者线程。

## 🔢 在工作者线程中动态执行脚本

工作者线程可以使用`importScripts()`加载和执行任意的脚本，这个方法可用于工作者线程内部的全局作用域，加载的脚本会按照加载顺序同步执行。
例如：

```js file:index.js
const worker = new Worker('./worker.js');

// importing scripts
// scriptA executes
// scriptB executes
// scripts imported
```

```js file:worker.js
console.log('importing scripts');
importScripts('./scriptA.js');
importScripts('./scriptB.js');
console.log('scripts imported');
```

```js file:scriptA.js
console.log('scriptA executes');
```

```js file:scriptB.js
console.log('scriptB executes');
```

`importScripts()`方法可以接收任意数量的脚本，浏览器在下载脚本的时候没有顺序的限制，但是在执行的时候会严格按照顺序执行。

```js file:worker.js
importScripts('./scriptA.js', './scriptB.js');
```

> [!tip]
> 在工作者线程中可以再创建子工作者线程，在有多个 CPU 核心的时候，使用多个子工作者线程可以实现并行计算。
> 但是使用多个子工作者线程的时候要考虑周全，确保这种投入确实是有收益的，毕竟运行多个子线程会有很大的计算成本。

<br />

## 🔢 处理工作者线程错误

如果工作者线程中抛出了异常，该异常是不会影响主线程执行的。

```js file:index.js
try {
	const worker = new Worker('./worker.js');
	console.log('no error');
} catch(e) {
	console.log('caught error');
}
```

上面的形式并不能捕获子线程的错误，不过子线程相关的错误依然会冒泡主线程的上下文中，可以通过`onerror`事件进行处理。

```js file:index.js
const worker = new Worker('./worker.js');
worker.onerror = console.log;
// ErrorEvent {message: "Uncaught Error: foo"}
```

## 🔢 工作者线程数据传输

使用工作者线程的时候，经常需要为它们提供某种形式的数据负载。工作者线程是独立的上下文，所以在上下文之间传递数据就会产生消耗（数据传输是有成本的）。

在支持多线程模型的语言中，可以使用锁、互斥量、和 volatile 变量。

> [!info]
> 工作者线程具有独立的上下文，有自己的 JS Runtime、调用栈，所以默认情况下它们之间不能共享对象。
> 例如主线程`var obj = { count: 1 }`不能直接给`Worker`使用，因为它们不在同一个内存中。
> 如果要进行数据的传递就需要进行跨线程，浏览器就需要：
> > - 复制数据
> > - 同步数据
> > - 转移内存
> > - 管理并发
>
> 这些操作都会消耗 CPU 和内存。
> 例如一个 500M 的数据，则会直接复制一份 500M 的数据，就会导致占用 CPU、占用内存、卡顿。
>
> 多线程语言中，都是共享同一份内存，但是会出现竞争问题，所以锁、互斥量、和 volatile 变量本质上都是为了解决竞争问题。
>
> JS 在设计之初就不是多线程语言，因此数据传输的消耗就比较高。

在 JS 中有三种方式在上下文之间进行数据的传输：结构化克隆算法、可转移对象、共享数组缓冲区。

## 🔢 1、结构化克隆算法

这种算法是浏览器在后台实现，不能直接进行调用。在通过`postMessage()`传递对象的时候，浏览器会遍历这个对象，并在目标上下文中生成一个副本。

以下是结构化克隆算法支持的类型：
- 除`symbol`之外所有的原始类型；
- `Boolean`对象
- `String`对象
- `Date`
- `RegExp`
- `Blob`
- `File`
- `FileList`
- `ArrayBuffer`
- `ArrayBufferView`
- `ImageData`
- `Array`
- `Object`
- `Map`
- `Set`

另外需要注意：
- 复制之后，源上下文对该对象的修改不会同步到目标上下文中；
- 该算法可以识别对象中的循环引用，不会无穷的遍历对象；
- 克隆`Error`对象、`Function`对象或者 DOM 节点会抛出异常；
- 对象属性描述符、`get()`、`set()`不会克隆，必要的时候使用默认值；
- 不会克隆对象的原型链；
- 不会克隆`RegExp.prototype.lastIndex`属性；

> [!tip]
> 克隆算法在对象比较复杂的时候会存在计算性消耗，实践中要尽可能避免过大、过多的复制。

<br />

## 🔢 2、可转移对象

可转移对象（transferable objects）可以把所有权从一个上下文转移到另一个上下文。在复制大量数据的情况下，这种方式非常有效。

只有下面的对象是可转移对象：
- `ArrayBuffer`
- `MessagePort`
- `ImageBitmap`
- `OffscreenCanvas`

`postMessage()`的第二个可选参数是一个数组，可以指定将哪些对象转移到目标上下文中。在遍历信息负载的时候，浏览器会遍历对象并检查对象的引用，对于可转移的对象则会进行转移，而不是复制它们。

```js file:index.js
const buffer = new ArrayBuffer(1024);

worker.postMessage(buffer, [buffer]);
```

```js file:index.js
const buffer = new ArrayBuffer(1024);

const data = {
  user: {
    name: "Tom"
  },
  buffer
};

// 嵌套对象也可以进行转移
worker.postMessage(data, [buffer]);
```

上面的示例中，`data`和`user`对象会进行复制，而`buffer`对象会进行转移。

## 🔢 3、SharedArrayBuffer

> [!info] 信息
> 由于 Spectre 和 Meltdown 的漏洞，所有的主流浏览器在 2018 年 1 月就禁用 SharedArrayBuffer。
> 在 2019 年的时候又重新启用，但是新增了安全机制，当服务器返回如下响应头时才可以使用这个对象：
> > ```http
> > Cross-Origin-Opener-Policy: same-origin
> > Cross-Origin-Embedder-Policy: require-corp
> > ```

这种方式是一种在不同线程之间可共享内存的数据结构，和`ArrayBuffer`不同，`ArrayBuffer`在复制和转移后得到的是一个独立的数据副本。

当`SharedArrayBuffer`被传递给`postMessage()`的时候，浏览器只会传递原始缓冲区的「引用」，结果就是不同的线程上下文「分别维护对同一个内存块的引用」,每个上下文都可以随意的修改这个缓冲区。

示例：

```js file:index.js
const worker = new Worker("./worker.js");
// 创建 1 字节缓冲区
const sharedArrayBuffer = new SharedArrayBuffer(1);
// 创建 1 字节缓冲区的视图
const view = new Uint8Array(sharedArrayBuffer);
// 父上下文赋值 1
view[0] = 1;

worker.onmessage = () => {
	console.log(`buffer value after worker modification: ${view[0]}`);
};
// 发送对 sharedArrayBuffer 的引用
worker.postMessage(sharedArrayBuffer);

// buffer value before worker modification: 1
// buffer value after worker modification: 2
```

```js file:worker.js
self.onmessage = ({ data }) => {
  const view = new Uint8Array(data);
  console.log(`buffer value before worker modification: ${view[0]}`);

  // 工作者线程为共享缓冲区赋值
  view[0] += 1;

  // 发送空消息，通知赋值完成
  self.postMessage(null);
};
```

当然在两个并行的线程中共享内存块存在竞争的风险，下面的示例验证了这一点：

```js file:index.js
// 创建包含 4 个线程的线程池
const workers = [];
for (let i = 0; i < 4; ++i) {
	workers.push(new Worker("./worker.js"));
}

// 在最后一个工作者线程完成后打印最终值
let responseCount = 0;
for (const worker of workers) {
	worker.onmessage = () => {
	  if (++responseCount == workers.length) {
		console.log(`Final buffer value: ${view[0]}`);
	  }
	};
}

// 初始化 SharedArrayBuffer
const sharedArrayBuffer = new SharedArrayBuffer(4);
const view = new Uint32Array(sharedArrayBuffer);

view[0] = 1;

// 把 SharedArrayBuffer 发给每个线程
for (const worker of workers) {
	worker.postMessage(sharedArrayBuffer);
}

// （期待结果为 4000001。实际输出类似于：）
// Final buffer value: 2145106
```

```js file:worker.js
self.onmessage = ({ data }) => {
  const view = new Uint32Array(data);

  // 执行 100 万次加操作
  for (let i = 0; i < 1e6; ++i) {
    view[0] += 1;
  }

  self.postMessage(null);
};
```

上面代码中每个线程都顺序执行了 100 万次加操作，每次都是读取共享的内存，然后写回数组索引中，在所有线程读写的过程中就会发生资源的竞争。

例如：
1. 线程 A 读取到的值为 1；
2. 线程 B 读取到的值为 1；
3. 线程 A 加 1 并将 2 写回数组；
4. 线程 B 仍然使用陈旧的数组值 1，同样把 2 写回数组；

为了解决竞争问题，可以使用`Atomics`对象让一个线程获取`SharedArrayBuffer`实例的锁，在执行完全部读写操作后，再允许另外一个线程执行操作。

```js file:index.js hl:6
self.onmessage = ({ data }) => {
  const view = new Uint32Array(data);

  // 执行 100 万次加操作
  for (let i = 0; i < 1e6; ++i) {
    Atomics.add(view, 0, 1);
  }

  self.postMessage(null);
};
```

## 🔢 线程池

因为启用工作者线程的代价很大，所以某些情况下可以考虑始终保持固定数量的线程活动。工作者线程在执行计算的时候会被标记为忙碌，直到它通知线程自己空闲了。

线程池中的线程数量并没有标准的答案，不过可以参考`navigator.hardwareConcurrency`属性返回系统可用的核心数量，因为不太可能知道每个核心的多线程能力，因此最好把这个数字作为线程池大小的上限。

## 🔢 共享工作者线程

共享工作者线程和专用工作者线程类似，但可以被多个信任的 Tab 标签访问。

共享工作者线程适用于开发者希望通过多个上下文共享线程减少计算消耗的情形。

从行为上讲，共享工作者线程可以视为专用工作者线程的一个拓展。线程的创建、选项、安全限制和`importScripts()`行为都是相同的。

```js file:index.js
const sharedWorker = new SharedWorker("./sharedWorker.js");
```

共享线程和专用线程的一个重要的差异在于`Worker()`会创建新实例，而`SharedWorker()`只会在相同的标识下不存在的情况才会创建新实例，如果存在和标识一致的共享线程，那么只会和存在的共享线程建立新的连接。

共享线程的标识源自于解析后的脚本 URL：
```js file:index.js
// 实例化一个共享工作者线程
// - 全部基于同源调用构造函数
// - 所有脚本解析为相同的 URL
// - 所有线程都有相同的名称
new SharedWorker("./sharedWorker.js");
new SharedWorker("./sharedWorker.js");
new SharedWorker("./sharedWorker.js");
```

类似的：

```js file:index.js
// 实例化一个共享工作者线程
// - 全部基于同源调用构造函数
// - 所有脚本解析为相同的 URL
// - 所有线程都有相同的名称
new SharedWorker('./sharedWorker.js');
new SharedWorker('sharedWorker.js');
new SharedWorker('https://www.example.com/sharedWorker.js');
```

另外还可以根据`name`来进行标识：

```js file:index.js
// 下面会创建两个共享线程，即使 URL 一致，但是 name 不一致
new SharedWorker("./sharedWorker.js", { name: "foo" });
new SharedWorker("./sharedWorker.js", { name: "bar" });
```

### 生命周期对比

专用工作者线程的生命周期：

| 事件               | 结果        | 事件发生后的线程数 |
| ---------------- | --------- | --------- |
| 标签页 1 执行 main.js | 创建专用线程 1  | 1         |
| 标签页 2 执行 main.js | 创建专用线程 2  | 2         |
| 标签页 3 执行 main.js | 创建专用线程 3  | 3         |
| 标签页 1 关闭         | 专用线程 1 终止 | 2         |
| 标签页 2 关闭         | 专用线程 2 终止 | 1         |
| 标签页 3 关闭         | 专用线程 3 终止 | 0         |

如上所述，脚本的执行次数、打开标签和运行线程是对等的关系。

下面是共享工作者线程的生命周期：

| 事件               | 结果                             | 事件发生后的线程数 |
| ---------------- | ------------------------------ | --------- |
| 标签页 1 执行 main.js | 创建共享线程 1                       | 1         |
| 标签页 2 执行 main.js | 连接共享线程 1                       | 1         |
| 标签页 3 执行 main.js | 连接共享线程 1                       | 1         |
| 标签页 1 关闭         | 断开与共享线程 1 的连接                  | 1         |
| 标签页 2 关闭         | 断开与共享线程 1 的连接                  | 1         |
| 标签页 3 关闭         | 断开与共享线程 1 的连接。没有连接了，因此终止共享线程 1 | 0         |

## 🔢 服务工作者线程

服务工作者线程是一种类似于浏览器中代理服务器的线程，可以拦截外出请求和缓存响应。这可以实现网页在没有网络的情况下正常使用，因为部分或全部页面可以从服务工作者线程缓存中提供服务。

对大多数开发者而言，服务工作者线程在两个主要任务上有用：
1. 充当网络请求缓存层
2. 启用推送通知

> [!info]
> 服务工作者线程最主要的场景是实现 Web 应用的离线功能，让用户在网络信号较差的地区可以继续使用应用程序。
> 当用户访问离线网页的时候，服务线程可以检查缓存中是否存在需要的资源，如果存在就直接提供，否则可以显示一个后备的内容。
> 所以，服务工作者线程也是 PWA（Progressive Web Application，渐进式 Web 应用） 的关键技术。

[具体实现，遇到再补充。]
