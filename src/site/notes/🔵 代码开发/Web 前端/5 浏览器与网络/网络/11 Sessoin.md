---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/5 浏览器与网络/网络/11 Sessoin/","dg-note-properties":{}}
---

---
---
Cookie 是保存在客户端的，虽然可以给服务器减少了很多的压力，但是某些情况下还是会出现麻烦。

例如下面的情况：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740467753850-b99b2ce3-6f07-410f-b542-3c28d26f2827.png)

如果按照上面的流程，客户端随便写一个别人的手机号，然后就能在 HTTP 的响应头中获取到短信验证码，这样验证码就没有任何的意义。

所以，有些敏感数据是万万不能发送给客户端的。

那么如何实现这个流程呢？

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740468278063-31d29a13-8423-4021-b3bf-ad352a3ed184.png)

正确的做法应该是服务器返回一个 Sessoin ID，同时服务器保存这个 Sessoin ID 和对应的内容。

<br/>tips
Info

Sessoin 和 Cookie 一样也是键值对的数据，只不过 Sessoin 是保存在服务器上。

<br/>

这样就可以避免 Cookie 的缺陷，当客户端获取验证码的时候，服务器收到请求，将真实的短信验证码通过短信的形式发送给指定的手机号，并在 Sessoin 中记录一条数据。然后将 Sessoin ID 通过 set-cookie 的头信息返回到客户端，客户端的 Cookie 会自动保存这个 ID。

当用户输入完验证并点击提交按钮的时候，浏览器会自动把 Sessoin ID 携带在请求头中发送到服务端。服务端得到 Sessoin ID 和验证码数据，然后去本地的 Sessoin 去验证一下即可。

下面是一道常见的面试题，Cookie 和 Sessoin 有什么区别：

1. Cookie 的数据保存在浏览器端。Session 的数据保存在服务器；
2. Cookie 的存储空间有限；Session 的存储空间不限；
3. Cookie 只能保存字符串；Session 可以保存任何类型的数据；
4. Cookie 中的数据容易被获取；Session 中的数据难以获取；
