---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/5 浏览器与网络/网络/12 JWT/","dg-note-properties":{}}
---

---
---
首先回顾一下之前讲 Cookie 登录的流程：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740463189999-45700f7a-95fe-4394-b997-c301bec6cb53.png)

- 客户端给服务端发起请求，将账号密码告诉服务端；
- 服务端返回一个“出入证”让客户端保存；
- 之后当每次客户端访问服务端的时候都需要携带上这个“出入证”，否则服务端就不认识客户端是谁；

那么接下来的问题就是：这个“出入证”里面存了啥？一种比较简单的办法就是里面直接存储用户信息的 JSON 串，但是这又会造成下面几个问题。

- 非浏览器环境（例如原生 APP、微信小程序），如何在“出入证”中记录过期的时间呢？
- 如果防止伪造的“出入证”呢？

JWT（JSON Web Token）就是为了解决这个问题的，其本质上就是一个字符串。它要解决的问题就是如何在互联网环境中提供一个统一的、安全的「令牌格式」。

因此 JWT 就是一个令牌格式而已，客户端拿到这个 JWT 后可以当在任何的地方，例如 Cookie、Storage，没有任何的限制。通常情况下，JWT 会作为响应体数据被返回，而不是放在 Header 的 set-cookie 中。

同样的，在传输的时候，开发人员也可以使用任何的方式进行传输。但是一般来说都会使用放在请求头中，例如:

```http
HTTP/1.1 200 OK
...
set-cookie: token=jwt令牌
...

{..., token: jwt令牌}
```

```http
GET /api/user/info HTTP/1.1
...
Authorization: jwt令牌
...
```

这样以来，服务器就可以接收到这个令牌了，然后会对这个令牌进行验证，就知道令牌是否有效。

完整的流程：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740464691285-da7cda5d-1a72-46d2-a0a5-1d5370d8fab0.png)

## 🔢 JWT 的组成
那么 JWT 是由什么组成的呢？它为什么可以防止被篡改呢？

为了保证数据的安全性，JWT 由三个部分组成：

- header: 令牌头部，记录了整个令牌的类型和签名算法；
- payload：令牌负荷，记录了保存的主体信息；
- signature：令牌签名，按照头部固定的签名算法对整个令牌进行签名，该签名的作用是保证令牌不被伪造和篡改；

它们三个组合而成的完整格式是：header.payload.signature。

例如，一个完整的 JWT 令牌如何：

```latex
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsaWNlbnNlIjoibWFkZSBieSB3YW5neSIsInJhbmRvbS0iOjE3Mzk3NzUzNjM3NzIsInVzZXJfbmFtZSI6IjE2ODkyMTY3OTg0Iiwic2NvcGUiOlsic2VydmVyIl0sImV4cCI6MTczOTg2MTc2MywidXNlcklkIjo1NDAwMDQsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIkNSTV9TTkFJTF9BRE1JTiJdLCJqdGkiOiJjOTg0ZDk2My1iNmMzLTRmZjMtOWRlMC0zYTFlODQwY2M4MjMiLCJjbGllbnRfaWQiOiJwaWcifQ.98E4Ocs35tdw12G9BV3JkoGiiRht3BxIrLmQ5ewYh30
```

- header: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
- payload: eyJsaWNlbnNlIjoibWFkZSBieSB3YW5neSIsInJhbmRvbS0iOjE3Mzk3NzUzNjM3NzIsInVzZXJfbmFtZSI6IjE2ODkyMTY3OTg0Iiwic2NvcGUiOlsic2VydmVyIl0sImV4cCI6MTczOTg2MTc2MywidXNlcklkIjo1NDAwMDQsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIkNSTV9TTkFJTF9BRE1JTiJdLCJqdGkiOiJjOTg0ZDk2My1iNmMzLTRmZjMtOWRlMC0zYTFlODQwY2M4MjMiLCJjbGllbnRfaWQiOiJwaWcifQ
- signature: 98E4Ocs35tdw12G9BV3JkoGiiRht3BxIrLmQ5ewYh30

## 🔢 header
它是令牌的头部，记录了整个令牌的类型和签名算法。它的格式就是一个 JSON 对象，例如：

```json
{
  "alg":"HS256",
  "typ":"JWT"
}
```

该对象记录了：

- `alg`：signature 部分使用的签名算法，通常有两个值；
    - HS256：一种对称加密算法，使用同一个秘钥对 signature 加密解密；
    - RS256：一种非对称加密算法，使用私钥签名，公钥验证；
- `typ`：整个令牌的类型，固定写`JWT`即可；

设置好 header 之后就可以生成 header 部分了，具体生成方式也非常的简单，就是把 header 部分使用 Base64 URL 进行编码即可。

<br/>tips
Base64 URL 不是一个加密算法，而是一种编码方式，它是在 Base64 算法的基础上对 +、-、/ 这三个字符做出特俗处理的算法。

而 Base64 是使用 64 个可打印字符来表示一个二进制数据。

<br/>

浏览器也提供了`btoa()`函数可以对数据进行编码：

```js
window.btoa(JSON.stringify({
  "alg":"HS256",
  "typ":"JWT"
}))
// 得到字符串：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

同时也有一个`atob()`函数可以对数据进行解码：

```js
window.atob("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9")
// 得到字符串：{"alg":"HS256","typ":"JWT"}
```

## 🔢 payload
这是 JWT 的主体信息，它依然是一个 JSON 对象，它可以包含以下内容：

```json
{
  "ss"："发行者",
  "iat"："发布时间",
  "exp"："到期时间",
  "sub"："主题",
  "aud"："听众",
  "nbf"："在此之前不可用",
  "jti"："JWT ID"
}
```

这些属性可以全写，也可以一个都不写，它只是一个规范，含义如下：

- ss：发行该 JWT 的是谁，可以写公司名字，也可以写服务名称；
- iat：该 JWT 的发放时间，通常写当前时间的时间戳；
- exp：该 JWT 的到期时间，通常写时间戳；
- sub：该 JWT 的目的；
- aud：该 JWT 是发放给哪个终端的，可以是终端类型，也可以是用户名称，随意一点；
- nbf：一个时间点，在该时间点到达之前，这个令牌是不可用的；
- jti：JWT 的唯一编号，设置此项的目的，主要是为了防止重放攻击（重放攻击是在某些场景下，用户使用之前的令牌发送到服务器，被服务器正确的识别，从而导致不可预期的行为发生）；

当用户登录成功之后，服务端就可以把用户的一些信息写入到这个对象中，例如用户 ID、昵称等等，例如：

```json
{
  "license":"made by wangy",
  "random-":1739775363772,
  "user_name":"16892167984",
  "scope":["server"],
  "exp":1739861763,
  "userId":540004,
  "authorities":["ROLE_USER","CRM_SNAIL_ADMIN"],
  "jti":"c984d963-b6c3-4ff3-9de0-3a1e840cc823",
  "client_id":"pig"
}
```

这个对象中`userId`、`user_name`、`authorities`都是服务端自定义的信息。

最终，payload 的部分和 header 一样都需要经过 Base64 URL 的编码。

```js
window.btoa(JSON.stringify({
  "license":"made by wangy",
  "random-":1739775363772,
  "user_name":"16892167984",
  "scope":["server"],
  "exp":1739861763,
  "userId":540004,
  "authorities":["ROLE_USER","CRM_SNAIL_ADMIN"],
  "jti":"c984d963-b6c3-4ff3-9de0-3a1e840cc823",
  "client_id":"pig"
}))
// 得到字符串：eyJsaWNlbnNlIjoibWFkZSBieSB3YW5neSIsInJhbmRvbS0iOjE3Mzk3NzUzNjM3NzIsInVzZXJfbmFtZSI6IjE2ODkyMTY3OTg0Iiwic2NvcGUiOlsic2VydmVyIl0sImV4cCI6MTczOTg2MTc2MywidXNlcklkIjo1NDAwMDQsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIkNSTV9TTkFJTF9BRE1JTiJdLCJqdGkiOiJjOTg0ZDk2My1iNmMzLTRmZjMtOWRlMC0zYTFlODQwY2M4MjMiLCJjbGllbnRfaWQiOiJwaWcifQ
```

## 🔢 signature
这一部分是 JWT 的签名，正是它的存在保证了 JWT 不能被篡改。

这部分的生成，是对前面两个部分的编码结果，按照头部指定的方式进行加密。

例如头部指定的加密方式是 HS256，前面两部分的编码结果是 eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsaWNlbnNlIjoibWFkZSBieSB3YW5neSIsInJhbmRvbS0iOjE3Mzk3NzUzNjM3NzIsInVzZXJfbmFtZSI6IjE2ODkyMTY3OTg0Iiwic2NvcGUiOlsic2VydmVyIl0sImV4cCI6MTczOTg2MTc2MywidXNlcklkIjo1NDAwMDQsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIkNSTV9TTkFJTF9BRE1JTiJdLCJqdGkiOiJjOTg0ZDk2My1iNmMzLTRmZjMtOWRlMC0zYTFlODQwY2M4MjMiLCJjbGllbnRfaWQiOiJwaWcifQ

则第三部分就是使用 HS256 算法对上面这段字符串进行加密，当然服务端需要使用一个密钥，例如密钥为 shhhhh。

```js
HS256(`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsaWNlbnNlIjoibWFkZSBieSB3YW5neSIsInJhbmRvbS0iOjE3Mzk3NzUzNjM3NzIsInVzZXJfbmFtZSI6IjE2ODkyMTY3OTg0Iiwic2NvcGUiOlsic2VydmVyIl0sImV4cCI6MTczOTg2MTc2MywidXNlcklkIjo1NDAwMDQsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIkNSTV9TTkFJTF9BRE1JTiJdLCJqdGkiOiJjOTg0ZDk2My1iNmMzLTRmZjMtOWRlMC0zYTFlODQwY2M4MjMiLCJjbGllbnRfaWQiOiJwaWcifQ`, "shhhhh")
// 得到：98E4Ocs35tdw12G9BV3JkoGiiRht3BxIrLmQ5ewYh30
```

最终这三个部分合并在一起就得到了一个完整的 JWT：

```latex
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsaWNlbnNlIjoibWFkZSBieSB3YW5neSIsInJhbmRvbS0iOjE3Mzk3NzUzNjM3NzIsInVzZXJfbmFtZSI6IjE2ODkyMTY3OTg0Iiwic2NvcGUiOlsic2VydmVyIl0sImV4cCI6MTczOTg2MTc2MywidXNlcklkIjo1NDAwMDQsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIkNSTV9TTkFJTF9BRE1JTiJdLCJqdGkiOiJjOTg0ZDk2My1iNmMzLTRmZjMtOWRlMC0zYTFlODQwY2M4MjMiLCJjbGllbnRfaWQiOiJwaWcifQ.98E4Ocs35tdw12G9BV3JkoGiiRht3BxIrLmQ5ewYh30
```

由于前面使用的密钥是保存在服务端的，因此客户端也就无法伪造出签名。

即使 header 和 payload 部分的数据被篡改了，但是到了服务器的时候会对其进行验证。

首先服务器会对 header 和 payload 使用同一个算法和密钥再次进行加密，然后讲加密的结果和 signature 部分进行对比。

如果对比的结果不一致表示这个 JWT 可能被篡改了，这个时候就可以提示客户端令牌过期，需要重新获取一个新的令牌等等。
