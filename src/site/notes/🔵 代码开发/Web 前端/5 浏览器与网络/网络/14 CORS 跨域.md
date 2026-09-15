---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/5 浏览器与网络/网络/14 CORS 跨域/","dg-note-properties":{}}
---

---
---
JSONP 并不是一个很好的解决跨域的方案，它至少存在下面两个严重的问题：

1、会打乱服务器的响应格式：JSONP 要求服务器返回一段 JS 代码。

2、只能完成 GET 请求：JSONP 的原理是通过`<script>`发起一个 GET 请求；

所以，CORS 是一种更好的解决跨域的方案。

## 🔢 概述
CORS 是基于 HTTP1.1 的一种跨域解决方案，全称是 Cross-Origin Resource Sharing，跨域资源共享。其总体思路是：如果浏览器要跨域访问服务器的资源，需要获得服务器的允许。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1748248542699-0d67ec59-a51d-4259-b147-98ae3431f801.png)

但是要知道，一个请求可以附带很多信息，从而会对服务器造成不同程度的影响。

例如有的请求只是获取一些新闻列表，有的请求则会改动服务器的数据。

针对不同的请求，CORS 规定了三种不同的交互模式，分别是：

- 简单请求；
- 预检请求；
- 附带身份凭证的请求；

## 🔢 简单请求
当 JS 发起一个请求的时候，浏览器首先会判断这个请求属于哪一种请求。

当请求同时满足以下条件的时候，浏览器会认为这是一个简单请求：

1、请求方法属于下面的一种：

- GET
- POST
- HEAD

2、请求头仅包含安全的字段，常见的安全字段如下：

- `Accept`
- `Accept-Language`
- `Content-Language`
- `Content-Type`
- `DPR`
- `Downlink`
- `Save-Data`
- `Viewport-Width`
- `Width`

3、如果请求头包含`Content-Type`，仅限下面的值之一：

- `text/plain`
- `multipart/form-data`
- `application/x-www-form-urlencoded`

如果同时满足上面的三个条件，则会被浏览器判定为简单请求。

示例：

```js
// 简单请求
fetch("http://crossdomain.com/api/news");

// 请求方法不满足要求，不是简单请求
fetch("http://crossdomain.com/api/news", {
  method: "PUT"
})

// 加入了额外的请求头，不是简单请求
fetch("http://crossdomain.com/api/news", {
  headers:{
    a: 1
  }
})

// 简单请求
fetch("http://crossdomain.com/api/news", {
  method: "POST"
})

// content-type 不满足要求，不是简单请求
fetch("http://crossdomain.com/api/news", {
  method: "POST",
  headers: {
    "content-type": "application/json"
  }
})

```

当浏览器判定某个请求是简单请求后，会发生以下的事情：

1、在请求头中添加`Origin`字段。

示例，在页面 http://my.com/index.html 中有以下代码造成了跨域：

```js
// 简单请求
fetch("http://crossdomain.com/api/news");
```

请求发出后，请求头会是下面的格式：

```http
GET /api/news/ HTTP/1.1
Host: crossdomain.com
Connection: keep-alive
...
Referer: http://my.com/index.html
Origin: http://my.com
```

`Origin`字段会告诉服务器，是哪个源地址在进行跨域请求。

2、服务器响应头中包含`Access-Control-Allow-Origin`。

当服务器收到请求后，如果允许该请求进行跨域访问，需要在响应头中添加`Access-Control-Allow-Origin`字段，该字段的值可以是：

- `*`：允许任何来源访问；
- 具体的域名：例如`http://my.com`，表示仅允许该域名访问；

> 服务端可以自己维护一个允许访问的源列表，如果请求的`Origin`包含在列表中，才响应`*`或者具体的源。
>

假设服务器响应了以下内容：

```http
HTTP/1.1 200 OK
Date: Tue, 21 Apr 2020 08:03:35 GMT
...
Access-Control-Allow-Origin: http://my.com
...

消息体中的数据
```

浏览器看到服务器允许自己访问资源后，高兴的像个两百斤的孩子，于是他就把响应顺利的交给 JS，然后完成后面的操作。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1748249644991-79c9f100-6aa9-4719-927a-706a95a4e504.png)

## 🔢 需要预检的请求
简单请求对服务器的威胁不大，所以允许使用上述的简单交互即可完成。

但是，如果浏览器不认为某个请求是简单请求，就会按照下面的流程进行：

1、浏览器发送预检请求，询问浏览器是否允许获取资源；

2、服务器返回允许；

3、浏览器发送真实的请求；

4、服务器完成真实的响应；

示例，在页面 http://my.com/index.html 中下面的代码造成了跨域请求：

```js
// 需要预检的请求
fetch("http://crossdomain.com/api/user", {
  method:"POST", // post 请求
  headers:{  // 设置请求头
    a: 1,
    b: 2,
    "content-type": "application/json"
  },
  body: JSON.stringify({ name: "张三", age: 18 }) // 设置请求体
});
```

浏览器发现它不是一个简单请求，则会按照下面的流程和服务器进行交互：

1、浏览器发送预检请求，询问浏览器是否允许。

```http
OPTIONS /api/user HTTP/1.1
Host: crossdomain.com
...
Origin: http://my.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: a, b, content-type
```

可以看出，这并不是 JS 发出的真实请求，请求中不包含我们的请求头，也没有请求体。

这是一个预检请求，它的目的就是询问服务器，是否允许后续的真实请求。

预检请求没有请求体，它包含了后续请求真实要做的事情。

预检请求有以下特征：

- 请求方法为`OPTIONS`；
- 没有请求体；
- 请求头中包含：
    - `Origin`：请求的来源，和简单请求含义一致；
    - `Access-Control-Request-Method`：后续的真实请求将使用的请求方法；
    - `Access-Control-Request-Headers`：后续的真实请求会改动的请求头；

2、服务器允许。

服务器收到预检请求后，可以检查预检请求中包含的信息，如果允许这样的请求，需要响应下面的消息格式：

```http
HTTP/1.1 200 OK
Date: Tue, 21 Apr 2020 08:03:35 GMT
...
Access-Control-Allow-Origin: http://my.com
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: a, b, content-type
Access-Control-Max-Age: 86400
...
```

对于预检请求，不需要响应任何的消息体，只需要在响应头添加：

- `Access-Control-Allow-Origin`：和简单请求一样，表示允许的源；
- `Access-Control-Allow-Methods`：表示允许的后续真实的请求方法；
- `Access-Control-Allow-Headers`：表示允许改动的请求头；
- `Access-Control-Max-Age`：告诉浏览器，多少秒内，对于同样的请求源、方法、头，都不需要再发送预检请求了；

3、浏览器发送真实的请求。

预检请求被服务器允许后，浏览器就可以发送真实的请求了，示例：

```http
POST /api/user HTTP/1.1
Host: crossdomain.com
Connection: keep-alive
...
Referer: http://my.com/index.html
Origin: http://my.com

{"name": "张三", "age": 18 }
```

4、服务器响应真实的请求。

```http
HTTP/1.1 200 OK
Date: Tue, 21 Apr 2020 08:03:35 GMT
...
Access-Control-Allow-Origin: http://my.com
...

{code: 0, message: "添加用户成功"}
```

可以看出，当完成预检请求后，后续的处理和简单请求相同。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1748251123648-67d45950-de4b-49b0-8e3f-b37185f45ae2.png)

## 🔢 附带身份凭证的请求
默认情况下，JS 发起 AJAX 的跨域请求并不会携带 Cookie，这样一来某些需要权限的操作就无法进行。不过，可以通过简单的配置就可以实现携带 Cookie 了。

```js
// xhr
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;

// fetch api
fetch(url, {
  credentials: "include"
})
```

这样一来，该跨域的 AJAX 请求就是一个携带身份凭证的请求了。

当一个请求携带 Cookie 的时候，无论它是简单请求还是预检请求，都会在请求头中添加 Cookie 字段。

而服务器响应的时候，需要明确告诉客户端，服务器允许这样的凭据。

告知的方式也非常简单，只需要在响应头中添加：`Access-Control-Allow-Credentials: true`即可。

对于一个携带身份凭证的请求，如果服务器没有明确告知，那么浏览器仍然视为跨域请求会拒绝。

另外还需要注意，对于携带身份凭证的请求，服务器不能设置`Access-Control-Allow-Origin: *`，这就是为什么不推荐使用`*`的原因。

## 🔢 获取响应头
在跨域访问的时候，JS 只能获取到一些基本的响应头，例如：`Cache-Control`、`Conent-Language`、`Conent-Type`、`Expires`、`Last-Modified`、`Pragma`，如果要访问其他的头信息，则需要服务器设置本响应头。

```http
Access-Control-Expose-Headers: authorization, a, b
```

以上表示服务器把允许浏览器访问的头放入了白名单中，这样 JS 就能够访问指定的响应头了。
