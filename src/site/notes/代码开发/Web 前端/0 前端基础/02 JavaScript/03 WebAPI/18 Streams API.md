---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/18 Streams API/","dg-note-properties":{}}
---

Streams API 是为了解决一个简单又基础的问题：Web 应用如何消费有序的小信息快块，而不是大块信息。

- 大块数据可能不会一次性可用，例如网络请求下载一个大的文件。网络传输中的内容都是以数据包的信息连续传输的，使用流式处理可以让数据一接收到立即使用；
- 大块数据可能需要分小部分处理。例如视频处理、数据压缩等都是可以分为小块数据进行处理，而不用等到全部加载到内存中再处理；

## 理解流
可以把数据想象成通过某种管道传输的液体。

Stream API 直接解决的问题是处理网络请求和读写磁盘。

tream API 定义了三种流：

- 可读流，数据（网络、文件等）从某个来源不断的进入流中，然后由消费者进行处理；
- 可写流，由提供者把数据一段一段的写进流中，流会把这些数据传递到某个目标（服务器、文件等）；
- 转换流，相当于一个中间人，一边接收输入的流，一边输出处理后的流；

## > 块、内部队列和反压
流的基本单位是块（chunk）。块可以是任意的数据类型，但通常是定型数组`TypedArray`。

每个块都是离散的流片段，可以作为一个整体来处理。更重要的是块不是固定大小的，也不一定按照固定的间隔到达。

上面提到的三种流都存在「入口」和「出口」的概念，由于数据进出的速率不同，可能会出现不匹配的情况。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776740625172-1fa18a55-52cb-4be7-8a1f-054516016d81.png)

1、流出口处理数据的速度比入口提供的数据速度更快。流出口经常空闲（也可能是流入口的效率低），但是只会浪费一点内存和计算资源，所以这种流的不平衡是可接受的。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776740913930-86a334a5-b949-4792-898b-e1ab48813b39.png)

2、流入和流出的速度相同，这就是最理想的状态。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776740988949-b5279ba2-8683-4ce2-a49b-54171435f638.png)

3、流入口提供数据的速度比出口处理数据的速度要快，这种不平衡是固有的问题，此时一定会出现数据的积压，流必须相应的做出处理。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776741230903-93522771-203f-48a8-a08a-2cbea0f2587f.png)

流的不平衡是常见的问题，但是流也提供了解决这个问题的工具。

所有的流都会为已经进入流，但尚未离开流的块提供一个内部队列。对于均衡流，这个内部队列会有零个或者少量排队的块，因为流出口的速度和流入口的速度几乎相等，这种流的内部队列所占用的内存相对较小。

如果块入列的速度快于出列的速度，则内部的队列会不断的增大。流不能允许内部的队列被无限的增大，因此它会使用「反压」通知流入口停止发送数据，直到队列大小降低到某个既定的阈值下。

这个阈值是由排队策略决定的，这个策略定义了内部队列可以占用的最大内存，也就是「高水位线」。

## 可读流
可读流是对底层数据源的封装，允许消费者通过公共的接口读取数据。

1、ReadableStreamDefaultController

示例：

```js
// 定义了一个生成器，每次等待 1s 就会返回一个递增的数字
async function* ints() {
  for (let i = 0; i < 5; i++) {
    yield await new Promise((resolve) =>
      setTimeout(() => resolve(i), 1000)
    );
  }
}
```

这个递增的数据可以通过「可读流控制器」传给可读流。访问控制器的方式就是创建一个`ReadableStream`的实例对象，并在构造函数中传入一个对象且带有`start()`方法：

```js
// 实例化
const readableStream = new ReadableStream({
  // 通过 start() 方法得到「控制器」
  start(controller) {
    console.log(controller); // controller 是 ReadableStreamDefaultController {} 的实例对象
  }
});
```

<br/>tips
`ReadableStreamDefaultController`就等同于：往流里面塞入数据的控制器。

<br/>

然后调用控制器的`enqueue()`方法把数据传给流，最后通过`close()`方法关闭流。

```js
const readableStream = new ReadableStream({
  async start(controller) {
    for await (const chunk of ints()) {
      // 往流里面塞入 chunk
      controller.enqueue(chunk);
    }
    // 关闭流
    controller.close();
  }
});
```

2、ReadableStreamDefaultReader

上面的示例中只是把值加入到了队列中，但是并没有把它们从队列中读取出来。所以需要一个`ReadableStreamDefaultReader`实例对象调用`getReader()`进行读取。

`readableStream`会返回一个`locked`属性用来判断这个读取器是不是被占用了。

```js
console.log(readableStream.locked); // false
const readableStreamDefaultReader = readableStream.getReader(); // 获得读取器
console.log(readableStream.locked); // true
```

然后消费者使用读取器调用`read()`方法可以读取数据：

```js
// 消费数据
(async function () {
  while (true) {
    // 读取数据
    const { done, value } = await readableStreamDefaultReader.read();
    if (done) {
      break;
    }
    console.log(value);
  }
})();
```

为什么需要`locked`进行读取器的锁定呢？想象下面这种情况：

```js
const reader1 = readableStream.getReader();
const reader2 = readableStream.getReader();
```

创建了两个读取器，就会产生几个问题：

- 数据到底给谁？
- 顺序怎么保证？
- 会不会重复 / 丢失？

因此浏览器规定，同一个时间只能有一个读取器，如果重复获取读取器则会产生错误。

最终在适当的时间释放读取器即可：

```js
reader1.releaseLock();
console.log(readableStream.locked); // false
```

然后就可以接着获取下一个读取器了。

## 可写流
可写流是底层数据槽（数据最终要被写到的地方）的封装，通过流的公共接口写入数据。

1、创建 WritableStream

依旧使用上面的生成器代码示例：

```js
async function* ints() {
  for (let i = 0; i < 5; i++) {
    yield await new Promise((resolve) =>
      setTimeout(() => resolve(i), 1000)
    );
  }
}
```

这些值可以通过可写流的公共接口写入流，在传给`WritableStream`的构造函数的对象中，通过实现`write()`方法可以获取「写入器写入」的数据：

```js
const writableStream = new WritableStream({
  write(value) {
    console.log(value);
  }
});
```

要把获得的数据写入流中，可以通过流的`getWriter()`方法获取`WritableStreamDefaultWriter`实例，`writableStream`同样的也会获得一个`locked`锁属性，确保只有一个写入器可以向流写入数据。

```js
console.log(writableStream.locked); // false
const writableStreamDefaultWriter = writableStream.getWriter(); // 写入器
console.log(writableStream.locked); // true
```

在向流写入数据之前，生产者必须确保写入器可以接收值：

```js
(async function(){
  for await (const chunk of ints()) {
    // 等待写入器可用
    await writableStreamDefaultWriter.ready;
    // 通过写入器写入数据
    writableStreamDefaultWriter.write(chunk);
  }
  writableStreamDefaultWriter.close();
})()
```

<br/>tips
`ready` 主要用于处理反压。当内部队列压力较大时，需要等待它变为可写状态后再继续写入。

<br/>

## 转换流
转换流用于组合可读流和可写流。数据块在两个流之间的转换是通过`transform()`方法完成的。

继续复用之前的生成器示例：

```js
async function* ints() {
  for (let i = 0; i < 5; i++) {
    yield await new Promise((resolve) =>
      setTimeout(() => resolve(i), 1000)
    );
  }
}
```

然后创建一个`TransformStream`的实例，并通过`transform()`方法把每个值翻倍：

```js
const { writable, readable } = new TransformStream({
  transform(chunk, controller) {
    // 对数据进行二次加工
    controller.enqueue(chunk * 2);
  }
});
```

向转换中的可读流和可写流传入数据和获取数据，和前面介绍的写入读取方式一致：

```js
// 创建读取器和写入器
const readableStreamDefaultReader = readable.getReader();
const writableStreamDefaultWriter = writable.getWriter();
```

```js
// 通过读取器读取数据
(async function (params) {
  while (true) {
    const { done, value } = await readableStreamDefaultReader.read();
    if (done) {
      break;
    }
    console.log(value);
  }
})();
```

```js
// 通过写入器写入数据
(async function(params) {
  for await (const chunk of ints()) {
    await writableStreamDefaultWriter.ready;
    writableStreamDefaultWriter.write(chunk);
  }
  writableStreamDefaultWriter.close();
})();
```

执行结果：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776760180518-8e254bbb-6063-4421-9222-502999961b9a.png)

## 通过管道连接流
流可以通过「管道」连接成一串。最常见的场景是使用`pipeThrough()`方法把可读流接入转换流。

从内部来看，可读流先把自己的值传递给转换流内部的可写流，然后执行转换，接着转换后的新值又在新的可写流上出现。

以下是一个对整数进行加倍处理的示例：

```js
async function* ints() {
  for (let i = 0; i < 5; i++) {
    yield await new Promise((resolve) =>
      setTimeout(() => resolve(i), 1000)
                           );
  }
}
```

```js
// 创建一个可读流
const integerStream = new ReadableStream({
  async start(controller){
    for await (const chunk of ints()) {
      // 可读流写入数据
      controller.enqueue(chunk);
    }
    // 写入完成
    controller.close();
  }
});

// 创建一个转换流
const doublingStream = new TransformStream({
  transform(chunk, controller) {
    // 对数据进行处理，然后再写入可读流
    controller.enqueue(chunk * 2);
  }
})
```

```js
// 进行管道连接，得到一个新的可读流
const pipedStream = integerStream.pipeThrough(doublingStream);

// 从新可读流中获取读取器
const pipedStreamDefaultReader = pipedStream.getReader();

// 消费数据
(async function () {
  while (true) {
    const { done, value } = await pipedStreamDefaultReader.read();
    if (done) {
      break;
    }
    console.log(value);
  }
})();
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776760180518-8e254bbb-6063-4421-9222-502999961b9a.png)

另外也可以使用`pipeTo()`将「可读流」连接到「可写流」，整个过程和`pipeThrough()`类似：

```js
const integerStream = new ReadableStream({
  async start(controller){
    for await (const chunk of ints()) {
      // 可读流写入数据
      controller.enqueue(chunk);
    }
    // 写入完成
    controller.close();
  }
});

const writableStream = new WritableStream({
  write(chunk) {
    console.log(chunk);
  }
});
const pipedStream = integerStream.pipeTo(writableStream);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776760180518-8e254bbb-6063-4421-9222-502999961b9a.png)

🔔 提示

当调用`pipeTo()`把可读流连接到可写流的时候，`pipeTo()`在读取到每个 chunk 后，自动调用目标可写流的`write(chunk)`写入数据。

因此以上代码中，不需要在调用写入器写入数据。

<br/>

## 实际场景
## > 大文件分块上传
核心思路：

```markdown
文件（File）
→ ReadableStream（分块读取）
→ 每一块调用上传接口（fetch）
```

示例：

```js
async function uploadFileInChunks(file, chunkSize = 1024 * 1024) {
  // 1MB 一块
  let offset = 0;

  const readableStream = new ReadableStream({
    // pull 方法每当流的内部队列不满时，会重复调用这个方法
    async pull(controller) {
      if (offset >= file.size) {
        controller.close();
        return;
      }

      // 切片
      const chunk = file.slice(offset, offset + chunkSize);

      // 转成 ArrayBuffer（可选）
      const buffer = await chunk.arrayBuffer();

      controller.enqueue(buffer);

      offset += chunkSize;
    }
  });

  // 读取并上传
  const reader = readableStream.getReader();

  let index = 0;

  while (true) {
    // 循环读取
    const { done, value } = await reader.read();
    if (done) break;

    // 上传文件
    await uploadChunk(value, index++);
  }

  console.log("上传完成");
}
```

```js
async function uploadChunk(chunk, index) {
  await fetch("/upload", {
    method: "POST",
    headers: {
      "Content-Type": "application/octet-stream",
      "X-Chunk-Index": index
    },
    body: chunk
  });

  console.log(`第 ${index} 块上传完成`);
}
```

## > 日志处理
核心思路：

```markdown
输入：日志流（Readable）
处理：
  1. 过滤错误日志
  2. 格式化
  3. 加时间戳
输出：写入文件（Writable）
```

链路规划：

```markdown
Readable（日志源）
→ pipeThrough（过滤）
→ pipeThrough（格式化）
→ pipeThrough（加时间戳）
→ pipeTo（写入文件）
```

示例：

```js
function createLogStream() {
  const logs = [
    "info: server started",
    "error: database failed",
    "info: request received",
    "error: timeout",
    "debug: cache hit"
  ];

  let i = 0;

  return new ReadableStream({
    start(controller) {
      // 定时循环写入数据
      const interval = setInterval(() => {
        if (i >= logs.length) {
          clearInterval(interval);
          controller.close();
          return;
        }

        controller.enqueue(logs[i++]);
      }, 500);
    }
  });
}
```

过滤错误日志：

```js
const filterErrorStream = new TransformStream({
  transform(chunk, controller) {
    if (chunk.startsWith("error")) {
      controller.enqueue(chunk);
    }
  }
});
```

格式化日志：

```js
const formatStream = new TransformStream({
  transform(chunk, controller) {
    const formatted = chunk.toUpperCase();
    controller.enqueue(formatted);
  }
});
```

添加时间戳：

```js
const timestampStream = new TransformStream({
  transform(chunk, controller) {
    const timestamp = new Date().toISOString();
    controller.enqueue(`[${timestamp}] ${chunk}`);
  }
});
```

写入文件：

```js
async function createFileWritable() {
  const handle = await window.showSaveFilePicker({
    suggestedName: "logs.txt"
  });
  return handle.createWritable();
}
```

串联管道：

```js
async function runPipeline() {
  const readable = createLogStream();
  const writable = await createFileWritable();

  await readable
    .pipeThrough(filterErrorStream)
    .pipeThrough(formatStream)
    .pipeThrough(timestampStream)
    .pipeTo(writable);

  console.log("日志处理完成");
}
```
