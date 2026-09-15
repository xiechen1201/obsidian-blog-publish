---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/07 Fetch API/","dg-note-properties":{}}
---

---
---
## 🔢 Fetch 概述
<br/>warning
⚠️ 注意

Fetch API 并不是 ES6 新增的，而是使用了 ES6 的 Promise API。

<br/>

Fetch API 就是用来进行 Ajax 请求的。

## 🔢 HMLHttpRequest 的问题
1、使用繁琐，所有的功能都全部集中在同一个对象上，容易写出混乱不易维护的代码；

2、采用传统的事件驱动的模式，无法适配新的 Promise API；

```js
var xhr = new XMLHttpRequest();

xhr.open('GET', 'https://v.api.aa1.cn/api/bilibili-rs/', true);

xhr.setRequestHeader('Content-Type', 'application/json');

// 定义事件处理程序，当请求状态变化时被调用
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    if (xhr.status === 200) {
      // 处理响应数据
      console.log('Response:', xhr.responseText);
    } else {
      // 处理错误情况
      console.error('Request failed with status:', xhr.status);
    }
  }
};

xhr.send();
```

## 🔢 Fetch 的特点
1、并非取代`XMLHttpRequest`，而是针对 AJAX 传统 API 的优化；

2、精细的功能划分：头部信息、请求信息、响应信息等分到不同的对象中去，更利于处理各种复杂的 AJAX 场景；

3、使用 Promise API 更利于异步代码的书写；

4、Fetch API 并非 ES6 的内容，属于 JavaScript 的 WebAPI；

## 🔢 基本使用
如何使用 Fetch API 进行网络请求呢？

```js
// 调用 fetch 函数即可
fetch(url, option);
```

参数：

- url：请求目标地址 URL，必填，字符串；
- option：配置选项，可选，请求配置内容，如果不填则使用默认配置；

options 请求配置对象：

- `method`：请求方式，默认为`GET`；
- `headers`：请求头信息；
- `body`：请求体信息，必须匹配`headers`中的`content-type`;
- `mode`：请求模式，用来解决跨域；
    - `cors`，默认值，会在请求头内加入`origin`和`referer`；
    - `no-cors`：不会发送`origin`和`referer`，跨域的时候可能出现问题；
    - `same-origin`：指示只允许同源请求，如果请求其他域，则会报错（也就是只能发送给当前域名，否则就会报错）；
- `credentials`：请求凭证
    - `same-origin`：默认值，请求同源地址携带 Cookie 的凭证；
    - `include`：请求任何地址携带 Cookie 的凭证；
    - `omit`：不发送任何凭证；
- `cache`：请求缓存，默认是`default`；

更多配置，详见 MDN：

[Window：fetch() 方法 - Web API | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/fetch)

[RequestInit - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/RequestInit)

<details class="lake-collapse"><summary id="u71b4e5f9"><span class="ne-text">为什么 credentials 属性和跨域有关联？</span></summary><p id="u0e75bf31" class="ne-p"><span class="ne-text">跨域的本质是“不同网站之间的隔离”，而</span><code class="ne-code"><span class="ne-text">credentials</span></code><span class="ne-text">会打破这种隔离（因为它携带用户身份）。</span></p><p id="ud8f32106" class="ne-p"><span class="ne-text">先想一个现实场景</span><span class="ne-text">👇</span></p><p id="ud8149c34" class="ne-p"><span class="ne-text">你登录了：</span></p><ul class="ne-ul"><li id="u380b2b26" data-lake-index-type="0"><code class="ne-code"><span class="ne-text">https://bank.com</span></code><span class="ne-text">（银行网站，有 cookie） </span></li></ul><p id="ue99e2966" class="ne-p"><span class="ne-text">然后你打开了一个恶意网站：</span></p><ul class="ne-ul"><li id="ud24d1c03" data-lake-index-type="0"><code class="ne-code"><span class="ne-text">https://evil.com</span></code></li></ul><p id="ua69694f3" class="ne-p"><span class="ne-text">那么恶意网站就会在</span><code class="ne-code"><span class="ne-text">evil.com</span></code><span class="ne-text">模拟你的请求：</span></p><pre data-language="javascript" id="EJDED" class="ne-codeblock language-javascript"><code>fetch(&quot;https://bank.com/transfer&quot;, {
  method: &quot;POST&quot;,
  credentials: &quot;include&quot;
})</code></pre><p id="u04b799d3" class="ne-p"><span class="ne-text">👉</span><span class="ne-text"> 如果浏览器允许：</span></p><ul class="ne-ul"><li id="ue27a6aa1" data-lake-index-type="0"><span class="ne-text"> cookie 被自动带上 </span></li><li id="udfc0370d" data-lake-index-type="0"><span class="ne-text"> 请求成功发出 </span></li></ul><p id="u2d41bdf3" class="ne-p"><span class="ne-text">💥</span><span class="ne-text"> 结果：</span></p><p id="ua8c5e78c" class="ne-p"><span class="ne-text">👉</span><span class="ne-text"> 钱直接被转走（CSRF 攻击）</span></p><p id="ud806c142" class="ne-p"><span class="ne-text">这就是网站跨域不处理的危险本质，所以浏览器必须进行介入。</span></p><p id="u8d0036a0" class="ne-p"><span class="ne-text"></span></p><p id="u96a13263" class="ne-p"><span class="ne-text" style="text-decoration: underline">浏览器采用的是：默认不携带凭证 + 必须服务端明确同意 的双保证。</span></p><p id="u642b2aec" class="ne-p"><span class="ne-text" style="text-decoration: underline"></span></p><p id="u88d17d03" class="ne-p"><span class="ne-text">默认只有同源的网站才会携带凭证</span><code class="ne-code"><span class="ne-text">credentials: &quot;same-origin&quot;</span></code><span class="ne-text">，这样 evil.com 就拿不到你的身份。</span></p><p id="udcfe8dc9" class="ne-p"><span class="ne-text">即使你设置了非同源也要携带凭证</span><code class="ne-code"><span class="ne-text">credentials: &quot;include&quot;</span></code><span class="ne-text">，那么浏览器也必须要求服务端明确说明同意携带凭证。</span></p><pre data-language="http" id="R4kvy" class="ne-codeblock language-http"><code>Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: https://evil.com</code></pre><p id="ub95dcf56" class="ne-p"><code class="ne-code"><span class="ne-text">Access-Control-Allow-Origin</span></code><span class="ne-text">这个字段不能设置为</span><code class="ne-code"><span class="ne-text">*</span></code><span class="ne-text">，因为如果设置为</span><code class="ne-code"><span class="ne-text">*</span></code><span class="ne-text">则等同于没有进行任何的限制。</span></p></details>

返回值：

使用`fetch()`方法后会返回一个 Promise 对象。当收到服务器的结果后，Promise 的状态为 fulfilled 的状态，数据为 Response 对象。当网络发送错误的时候（或者其他导致无法完成交互的错误），Promise 的状态为 rejected 的状态，数据为错误对象。

使用案例：

```js
fetch('https://v.api.aa1.cn/api/bilibili-rs/')
  .then(response => {

    // 检查响应状态
    if (!response.ok) {
      throw new Error('Network response was not ok ' + response.statusText);
    }
    // 解析 JSON 响应体
    return response.json();
  })
  .then(data => {
    // 处理数据
    console.log('Data:', data);
  })
  .catch(error => {
    // 处理错误
    console.error('There was a problem with the fetch operation:', error);
  });
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1722563344106-7b93ef93-04a2-4145-b7b3-092bfc439cbb.png)

## 🔢 Response 对象
上面说了，当`fetch()`请求成功返回一个状态为 fulfilled 的 Promise 对象，值为`Response`对象，下面看看该对象都有哪些属性：

- `ok`：布尔值，当状态码在 200-299 之间时为`true`，否则为`false`；
- `status`：响应状态码；
- `statusText`：响应状态文本。例如，对于状态码 404，`statusText`通常是 “Not Found”；
- `headers`：服务端响应头信息。可以使用`headers.get('Content-Type')`获取特定的响应头；
- `url`：响应的地址，表示为经过重定向后最终获得的 URL 地址。
- `redirected`：布尔值，当响应地址和请求地址不同时为`true`，否则为`false`；
- `text()`：用于处理文本格式的响应。它从响应中获取文本流，将其读完，然后返回一个被解决为`String`对象的 Promise。
- `json()`：返回一个`Promise`对象，解析响应体为 JSON 对象
- `blob()`：返回一个`Promise`对象，解析响应体为`Blob`对象
- `arrayBuffer()`：返回一个`Promise`对象，解析响应体为`ArrayBuffer`对象
- `formData()`：返回一个`Promise`对象，解析响应体为`FormData`对象
- `redirect()`：可以用于重定向到另外一个 URL 地址，他会创建一个新的`Promise`，以解决来自重定向 URL 的响应。

使用示例：

```js
getBilibili();
async function getBilibili() {
  let result = await fetch(URL);

  console.log(result);

  // 将内容解析为纯文本
  // let text = await result.text()
  // console.log(text);

  // 将内容解析为 JSON
  let json = await result.json();
  console.log(json);
}
```

`response.json()`等方法为什么是`Promise`对象？

是因为`response.json()`需要时间来读取并解析响应体（body）为 JSON 格式，尤其是在处理较大或复杂的数据时。使用`Promise`可以在解析完成后继续处理数据，而不会阻塞主线程。

`response.json()`的工作原理：

1. 响应体的流式读取:
    1. HTTP 响应体在接收到时是一个字节流。
        1. `response.json()`首先将字节流读取到内存中。
    2. 字节流到字符串的转换:
        1. 读取的字节流被转换为字符串。
        2. 这是一个潜在的耗时操作，特别是对于大响应体。
    3. 字符串到 JSON 的解析:
        1. 转换后的字符串通过`JSON.parse()`解析为 JavaScript 对象。
        2. 解析操作可能会因数据复杂性而耗时。

<br/>

## 🔢 Request 构造函数
`fetch(url, options)`内部会将两个参数进行合并并封装为一个`Request`对象，因此我们可以直接传递一个`Request`对象。

```js
new Request(url, options);
```

使用`Requesut`对象的好处是，某些时候要得到一个通用的请求方式，可以使用函数创建一个`Request`对象。

```js
function getRequstInfo() {
  const url = 'https://v.api.aa1.cn/api/bilibili-rs/';
  const req = new Request(url, {});
  return req;
}
```

尽量保证每次请求都是一个全新的`Request`对象。

<br/>

```js
// Bad

let request = null;

if (request) {
  // ...
} else {
  request = getRequstInfo();
}
```

如果在进行 POST 请求时需要发送一个请求体，而请求体的数据量较大或包含二进制文件，那么请求体可能会以流的形式处理。在这种情况下，复用 Request 对象可能会导致流状态被意外复用，例如，之前的文件上传进度可能会被保留，这可能会导致数据传输错误或不完整。

```js
// Good

let request = null;

if (request) {
  // 克隆一个全新的 request 对象，配置保持一致！
  request = request.clone();
} else {
  request = getRequstInfo();
}
```

## 🔢 Response 构造函数
`Response`对象也可以进行手动的创建，绝大多数的时候都不需要自己手动的创建，一般测试的时候可以自己手动的创建。

```js
let resp = new Response('{"name":"张三"}', {
  // response 对象的一些配置
});

async function getJSON(resp) {
  return await resp.json();
}
```

更多配置，详见 MDN：

[Response() - Web API | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Response/Response)

## 🔢 Headers 对象
在`Request`和`Response`对象中，`headers`属性都是一个`Headers`对象。同时，我们可以直接传递一个`Headers`对象，好处在于可以实现复用。

如果你在控制台中展开`headers`属性，你会发现什么都看不到：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1722563470975-8cdfa7a4-41c5-4950-80c5-c08fa51e289f.png)

我们需要使用相关的方法进行获取：

- `has(key)`：检查头中是否存在指定的属性。
- `get(key)`：获取头中指定属性的值。
- `set(key, value)`：设置头中指定属性的值。
- `append(key, value)`：在头中追加指定属性的值。
- `delete(key)`：删除头中指定属性的值。
- `forEach(callback)`：遍历头中的所有属性。
- `keys()`：返回头中所有属性的 Key 的迭代器。
- `values()`：返回头中所有属性的 Value 的迭代器。
- `entries()`：返回头中所有属性的 Key 和 Value 的迭代器。

示例：

```js
fetch('https://v.api.aa1.cn/api/bilibili-rs/').then((res) => {
  console.log(res);
  console.log(res.headers.has('content-type'));
  console.log(res.headers.get('content-type'));

  res.headers.forEach((value, key) => console.log(key, value));

  // res.headers.entries() 返回的是一个迭代器！
  for (const iterator of res.headers.entries()) {
    console.log(iterator);
  }
});
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1722564734800-ce1c5d6b-912e-4118-8722-8d6dc2073577.png)
