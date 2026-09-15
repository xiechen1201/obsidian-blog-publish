---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/02-cookie/","dg-note-properties":{}}
---

## 🔢 Cookie 和服务器的交互
HTTP Cookie 通常也叫作 Cookie，最初用于在客户端存储会话信息。

这个规范要求服务器在响应 HTTP 请求时，通过发送 Set-Cookie 字段用来设置 Cookie 到客户端，用来标识信息等。

```http
HTTP/1.1 200 OK
Content-type: text/html
Set-Cookie: name=value
Other-header: other-header-value
```

浏览器会存储这些会话信息，并在之后的每个请求中都会通过 HTTP 头部 Cookie 再将它们发回服务器。

键名和值在发送时都会经过 URL 编码。

```http
GET /index.jsl HTTP/1.1
Cookie: name=value
Other-header: other-header-value
```

## 🔢 限制
1. Cookie 是与特定域绑定的。设置 Cookie 后，它会与请求一起发送到创建它的域。这个限制能保证Cookie 中存储的信息只对被认可的接收者开放，不被其他域访问。
2. Cookie 在客户端为防止被恶意利用是有大小限制的。大多数浏览器对 Cookie 的限制是不超过 4096 字节，上下可以有一个字节的误差。

## 🔢 Cookie 的构成
Cookie 在浏览器中是由以下参数构成的：

- name:  Cookie 的名称，不区分大小写，所以 myCookie 和 MyCookie 是同一个名称；
- value：存在 Cookie 里的值；
- domain：Cookie 有效的域；
- path: 请求 URL 中包含这个路径才会把 Cookie 发送到服务器；
- 过期时间：表示何时删除 Cookie 的；
    - 可以是 expires 值为时间对象（以 UTC 或 GMT 时间）；
    - 或者 max-age 值为具体的秒数；
    - 如果不设置过期时间 Cookie 只有在「浏览器关闭后」才会被清除！
- secure：设置之后，只在使用 SSL 安全连接（也就是 HTTPS 协议）的情况下才会把 Cookie 发送到服务器；

以上参数必须使用「分号+空格」分割：

```plain
Set-Cookie: name=value; expires=Mon, 22-Jan-07 07:10:24 GMT; domain=wrox.com
```

<br/>warning
⚠️ 注意

域、路径、过期时间和 secure 标志用于告诉浏览器什么情况下才能在请求中包含 Cookie。 这些参数并不会随请求发送给服务器，实际发送的只有 Cookie 的键/值对。

<br/>

## 🔢 JavaScript 中的 Cookie
在 JavaScript 中可以使用`document.cookie`来操作 Cookie。

> 使用这种方式操作 Cookie 存在一些问题，首先就是不方便！
>
> 现在官方也推出了 Cookie Store API，详见：[https://developer.mozilla.org/zh-CN/docs/Web/API/Cookie_Store_API](https://developer.mozilla.org/zh-CN/docs/Web/API/Cookie_Store_API)
>

取值：

`document.cookie`返回包含页面中所有有效 Cookie 的字符串（根据域、路径、过期时间和安全设置），以分号分隔。（所有名和值都是 URL 编码的，因此必须使用`decodeURIComponent()`解码。）

```js
console.log(document.cookie);
// name1=value1;name2=value2;name3=value3
```

<u></u>

设值：

在设置值时，可以通过`document.cookie`属性设置新的 Cookie 字符串。设置 Cookie，与 HTTP 响应头中 Set-Cookie 属性的格式是一样的。

在所有这些参数中，只有 Cookie 的名称和值是必需的。

```js
document.cookie = "name=value; expires=expiration_time; domain=domain_name; path=domain_path; secure"
```

设置过期时间：

```js
let day = new Date();
let d = day.getDate() + 1;
day.setDate(d);

document.cookie = "name=xiaoming; expires=" + day; // 一天后过期
// 或者
document.cookie = "name=xiaoming; max-age=20"; // 20 秒后过期
```

删除值：

如果要想删除一个`cookie`只需要将这个`cookie`的有效期设置为过去时间就可以了

```js
document.cookie = "name=xiaoming; expires=" + new Date(0);
document.cookie = "name=xiaoming; max-age=-1";
```

封装一个工具类：

```js
class CookieUtil {
  static get(name) {
    let cookieName = `${encodeURIComponent(name)}=`,
        cookieStart = document.cookie.indexOf(cookieName),
        cookieValue = null;

    if (cookieStart > -1){
      let cookieEnd = document.cookie.indexOf(";", cookieStart);
      if (cookieEnd == -1){
        cookieEnd = document.cookie.length;
      }
      cookieValue = decodeURIComponent(document.cookie.substring(cookieStart, cookieEnd));
    }

    return cookieValue;
  }

  static set(name, value, expires, path, domain, secure) {
    let cookieText = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`
    if (expires instanceof Date) {
      cookieText += `; expires=${expires.toGMTString()}`;
    }
    if (path) {
      cookieText += `; path=${path}`;
    }
    if (domain) {
      cookieText += `; domain=${domain}`;
    }
    if (secure) {
      cookieText += "; secure";
    }
    document.cookie = cookieText;
  }

  static unset(name, path, domain, secure) {
    CookieUtil.set(name, "", new Date(0), path, domain, secure);
  }
};

// ## 使用案例

// 设置 cookie
CookieUtil.set("name", "Nicholas");
CookieUtil.set("book", "Professional JavaScript");

// 读取
alert(CookieUtil.get("name")); // "Nicholas"
alert(CookieUtil.get("book")); // "Professional JavaScript"

// 删除
CookieUtil.unset("name");
CookieUtil.unset("book");

// 设置有路径、域和过期时间的
CookieUtil.set("name", "Nicholas", "/books/projs/", "www.wrox.com", new Date("January 1, 2010"));

// 删除刚刚设置的
CookieUtil.unset("name", "/books/projs/", "www.wrox.com");

// 设置安全
CookieUtil.set("name", "Nicholas", null, null, null, true);
```
