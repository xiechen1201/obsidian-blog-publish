---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/5 浏览器与网络/网络/04 HTTP 请求/","dg-note-properties":{}}
---

---
---
## 🔢 HTTP 协议
HTTP 协议是专为浏览器和服务器通信定制的一种协议，HTTP 会通过一个可靠的连接来交换信息，是一个无状态的请求/响应协议。

<br/>tips
Info

无状态：指没有记忆，例如第一次请求响应后，再一次进行请求时是不知道上一次的内容的，请求和请求之间没有关联性。

<br/>

只要客户端和服务器知道如何处理数据内容，任何类型的数据都可以通过 HTTP 发送，客户端以及服务器指定使用合适的 MIME 内容类型。

HTTP 报文长什么样子？

报文就是在客户端与服务器之间发送的数据块，这些数据块以一些文本的元信息（meta-information）开头，描述了报文的内容及含义，后面跟着可选的数据部分，这些报文在客户端、服务器和代理之间流动，所以 HTTP 报文的发送也叫报文流。

简单理解就是：客户端和服务端之间的数据传递，告诉服务端本次请求的目的，服务端返回数据，传输的信息就是报文！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1681805355659-8a90babe-dbb0-4248-aa27-7155eedd3c1f.png)

每次 HTTP 报文包含一个客户端请求（Request）和服务端响应（Response）。HTTP 协议是基于 TCP/IP 协议来通信的（先建立连接才能发起请求）。

<br/>tips
Info

TCP/IP 是一种网络协议的集合，它定义了计算机如何在互联网上进行通信。你可以把它看作是一个规则和标准的组合，帮助不同设备通过网络互相“对话”。

<br/>

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740034235419-ce809cdf-1f85-4e98-a4f7-4592989e861c.png)

客户端和服务器之间还可能存在代理，HTTP 请求的流向过程。

服务器处理完客户端的请求，并收到客户端的应答后，即断开连接（节省传输时间）；

以下是 HTTP 报文的组成部分：

1、对报文进行描述的起始行。

2、包含属性的首部/头部（Header）。

3、包含数据的主体（Body），可选项。

![请求报文](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658476885718-8c1eb21b-7ad6-4a4e-a8fa-bb8e0b33b5e4.png)

请求报文的简化版：

![报文的基本格式](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658476917970-03e83aad-55d5-4a05-b73e-f9b7e1b660a5.png)

![响应报文](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658476886712-e145d480-ebdb-4466-ab32-b226f6a1a946.png)

所以，一个完整的 HTTP 报文包括：

- 请求：请求头、请求体；
- 响应：响应头、响应体；

开发中最常接触的就是 GET 和 POST 的请求方法，下面是这两种方法的请求体：

![POST](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658477013114-9085f044-be68-4f27-8236-56b42d6e5c57.png)

![GET](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658477012830-d1a80180-36c7-441a-9585-a9541d416cff.png)

一般 POST 请求都是 form-data 表单数据，GET 请求都是查询字符串参数。

不管是 GET 的 Query Param 还是 POST 的 form-data，其实都是通过字符串拼接进行数据传递！只是 GET 是在 URL 能看到，而 POST 不能在 URL 上看到！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1681954684789-d9d788c8-b816-4403-9ecb-0e3174c1d9df.png)

下面是一个完整的 POST 请求的示例内容：

```http
POST /submit-form HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Content-Type: application/x-www-form-urlencoded
Content-Length: 39
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

name=John+Doe&email=john.doe%40example.com&message=Hello+World
```

```http
HTTP/1.1 200 OK
Date: Wed, 21 Oct 2020 07:28:00 GMT
Server: Apache/2.4.7 (Unix)
Content-Type: text/html; charset=UTF-8
Content-Length: 131
Connection: close

<html>
<head>
  <title>Form Submission</title>
</head>
<body>
  <h1>Thank you, John Doe!</h1>
  <p>We have received your message: Hello World</p>
</body>
</html>
```

## 🔢 请求方式
HTTP 的请求方式是请求头中的第一个单词，用于描述本次请求的动作类型，不同的请求方法只是包含了不同的语义，但不是强制的规定。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740035035176-429a6e17-86f8-4722-9001-9987f1bd772e.png)

例如我们也可以自定义一些请求方式：

```js
fetch('https://www.baidu.com', {
  method: 'heiheihei', // 告诉百度，我这次请求是来嘿嘿嘿的
});
```

虽然百度服务器无法理解“heiheihei”这个请求方式，但是这个请求是可以正常发送到对方服务器的。

在实践中，客户端和服务器之间达成了一种共识，规定了一些常用的请求方法：

- GET，表示向服务器获取资源。业务数据在请求行中，无须请求体；
    - 也可以有请求体，但是一般不这么操作；
- POST，表示向服务器提交信息，通常用于产生新的数据，比如注册，业务数据在请求体中；
- PUT，表示希望修改服务器的数据，通常用于修改。业务数据在请求体中；
- DELETE，表示希望删除服务器的数据。业务数据在请求行中，无须请求体；
- OPTIONS，发生在跨域的预检请求中，表示客户端向服务器申请跨域提交；

```http
OPTIONS /path/to/resource HTTP/1.1
Host: example.com
```

```http
HTTP/1.1 200 OK
Allow: GET, POST, PUT, DELETE
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Content-Type: text/html; charset=UTF-8
```

- TRACE，回显服务器收到的请求，主要用于测试和诊断；

```http
TRACE /path/to/resource HTTP/1.1
Host: example.com
User-Agent: MyBrowser/1.0
```

```http
HTTP/1.1 200 OK
Content-Type: message/http

TRACE /path/to/resource HTTP/1.1
Host: example.com
User-Agent: MyBrowser/1.0
```

- CONNECT，用于建立连接管道，通常在代理场景中使用，网页中很少用到；
- HEAD，与 GET 类似，但是不返回响应体。可以用来检查某个资源（例如一个网页）的是否存在，而不需要下载整个文件，通过检查响应状态码来判断资源是否可用；

Tip

不同的请求方式是为了分清楚不同请求的目的，但是并不代表用了 POST 就一定要修改数据，用 GET 就不能修改资源。

<br/>

## 🔢 GET 和 POST
GET 请求和 POST 请求是浏览器和服务器约定的一种规则，这种规则导致了两种请求在 Web 中的区别：

- GET 请求只能传递 ASCII 数据，遇到非 ASCII 数据需要进行编码。POST 请求则没有限制；
- GET 请求的数据都附带在 path 中，能够通过分享地址完整的重现页面。而 POST 请求的数据都携带在请求体中；
- GET 请求的传递信息量有限，适合传递少量数据（请求行和请求头是有大小限制的，是浏览器规定的）。POST 请求的传递信息量是没有限制的，适合传输大量数据；
- GET 请求的地址可以被保存为浏览器书签，POST 不可以；
- GET 请求的数据会进行缓存，POST 不会；
- 刷新页面时，若当前的页面是通过 POST 请求得到的，则浏览器会提示用户是否重新提交。若是 GET 请求得到的页面则没有任何提示；

## 🔢 状态码
HTTP 状态码可以分为以下 5 类：

| 1xx | 信息，服务器收到请求，需要请求者继续执行操作 |
| --- | --- |
| 2xx | 成功，操作被成功接收并处理 |
| 3xx | 重定向，需要进一步的操作以完成请求 |
| 4xx | 客户端错误，请求包含语法错误或无法完成请求 |
| 5xx | 服务器错误，服务器在处理请求的过程中发生了错误 |

下面是常见的具体状态码：

1、304 （未修改）自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。

> 重定向：重定向简单说就是跳转页面。例如点击一个超链接，点击后跳转页面，但是这个页面不进行长时间的停留，自动有跳转到另外一个页面，一般来说跳转就是重定向。
>
> 回到 304 状态码，当客户端请求服务器资源的时候，如果发现资源没有进行更改，那么就会重定向到浏览器缓存进行获取资源。
>

![第一次请求页面](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658738482891-f7b8a116-d75d-47f6-ba95-b71e13e2e206.png)

![第二次请求页面](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658738486649-d87dd6d6-6f96-4483-b8ed-450531869258.png)

我们可以看到第一次请求成功 200 的时候响应头中存在 ETag（资源的唯一标识）和 Last-Modified（资源最后修改的时间）两个属性。

> 每一次更改服务器的资源都会产生新的 ETag 和 Last-Modified！！！
>

第二次请求的时候返回 304 表示重定向，表示可以从缓存中的拿去页面（如果页面更新会再次返回 200 和新的 ETag 和 Last-Modified ）。

服务器怎么知道缓存中的资源是否和服务器中资源一样？

看请求头中的 If-None-Match 和 If-Modified-Since (上一次请求资源接收到的 ETag 和 Last-Modified 是否一致)，然后到服务端进行对比告诉浏览器从缓存中拿取资源。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1681973158406-7d5c88aa-c7fc-4e91-bd41-3ff541deffb9.png)

2、302 页面重定向

3、403 拒绝请求（例如服务器关闭时，或者没有权限的时候，或者访问局域网）

4、404 找不到相应的资源

5、500 服务器发生不可预测的错误

6、503 服务器当前不能处理客户端请求

更多状态码：

[HTTP状态码.docx](https://www.yuque.com/attachments/yuque/0/2025/docx/209060/1740752081831-4cda50d3-cfc1-4cbb-b2ef-07fc5f98a54d.docx)
