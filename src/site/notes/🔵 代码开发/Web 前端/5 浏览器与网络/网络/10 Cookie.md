---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/5 浏览器与网络/网络/10 Cookie/","dg-note-properties":{}}
---

---
---
讲 Cookie 之前，我们先讲一个案例。

假设服务器有一个接口，可以通过这个接口向服务器添加一个管理员。但是添加肯定不是所有人都可以操作的，那么服务器如何知道请求接口的人是否具有权力呢？那就是只有登录过的管理员才能进行操作。

可问题是，客户端和服务器之间使用的是 HTTP 协议，HTTP 协议是无状态的。

<br/>tips
Info

无状态就是服务器不知道这一次请求人，跟之前登录成功请求的人是不是同一个人。

<br/>

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740036575788-a82cb8c3-a9b6-4785-a135-3fd8abec0b24.png)![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740036875874-67dfe2b2-6dee-4606-b044-8bbb04b49112.png)

由于 HTTP 协议是无状态的，服务器忘记了之前的所有请求。它就无法确定这一次请求的客户端，是否是之前登录成功的那个客户端。

于是，服务器就想了一个办法，它会按照下面的流程来认证客户端的身份：

- 客户端登录成功后，服务器给客户端一个出入证；
- 后续客户端每次请求的时候，都必须带上这个出入证；

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740037133155-262dc4be-cb66-4b5b-a352-fc02280eb50a.png)

这样就可以轻松的识别客户端的身份了。

但是一个新的问题来了，用户不可能只在一个网站登录，于是服务器会收集到各个网站的出入证，所以就需要客户端有一个类似卡包的东西，并具备以下的功能：

- 可以存放多个出入证。这些出入证来自于不用的网站，也可能是一个网站可能有多个出入证，分别用于出入不同的地方；
- 能够自动出示出入证。客户端访问服务器的时候能够把对应的出入证自动携带过去；
- 正确的出示出入证。确保客户端不要把其他网站的出入证发送过去；
- 管理出入证的有效日期。客户端要能自动发送哪些出入证已经过期，并进行移除；

能满足以上这些需求的就是 Cookie，Cookie 就类似于一个卡包，专门存放各种出入证，并有一套机制来自动管理这些出入证。

## 🔢 Cookie 的组成
Cookie 是浏览器中特有的概念，每一个 Cookie 就相当于某个网站的一个卡片，它记录了下面的信息：

- key，键；
- value，值；
- domain，域名，表示当前 Cookie 属于哪个域名网站；
- path，路径，表示当前 Cookie 属于网站的哪个路径下；
- secure：是否使用安全传输，例如 HTTPS；
- expire：过期时间，表示当前 Cookie 在什么时候过期；

当浏览器向服务器发送一个请求的时候，浏览器会在本地瞄一眼自己的卡包，看看哪些卡包是符合条件的，并把符合条件的出入证携带到请求头中：

- Cookie 的 domain 和本次请求的域名是否匹配；
    - 例如 Cookie 的 domain = yuanjin.tec，则 yuanjin.tech、www.yuanjin.tech、blogs.yuanjin.tech 都是匹配的；
- Cookie 的 domain 和本次请求的 path 是否匹配；
    - / 表示根路径，即所有的 path 都匹配；
- 验证 Cookie 是否是安全传输，如果 secure 为`true`，则请求的协议必须是 HTTPS；
- 验证 Cookie 是否已经过期；

如果一个 Cookie 满足上面的所有条件，则会把这条 Cookie 携带中请求头中，例如访问百度：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740038969343-f20d2c9a-e64f-49b8-85a7-e539615c2091.png)

请求百度的时候会携带非常多的 Cookie，每个 Cookie 之间使用`; `分隔。

## 🔢 如何设置 Cookie
由于 Cookie 是保存在浏览器的，同时很多的出入证都是服务器颁发的，所以 Cookie 的设置有两种方式：

- 服务器的响应头；
    - 这是非常普通的方式，如果服务器要在客户端设置一个出入证，则响应头信息中会包含一个 Set-cookie 的字段；
    - ![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1740039236308-4d968835-103b-484e-bbbf-3bf0241cb5b1.png)
- 客户端自行设置；
    - 这种模式比较少见，不过也可能发生，例如用户关闭了某个广告，并选择“不在弹出”，此时就可以通过浏览器的 JS 代码保存到 Cookie。后续请求服务器的时候，服务器会读取是否存在这个 Cookie 并且不再发送广告；

## 🔢 服务器设置 Cookie
上面说了，服务器设置 Cookie 的方式就是在响应头中增加一个 Set-cookie 的字段，格式如下：

```latex
set-cookie: cookie1
set-cookie: cookie2
set-cookie: cookie3
...
```

通过这种模式，就可以在一次响应中设置多个 Cookie 了。

其中，每个 Cookie 的格式如下：

```latex
键=值; path=?; domain=?; expire=?; max-age=?; secure; httponly
```

每个 Cookie 的键和值是必须的，其余都是可选的，并且顺序不限。这样浏览器看到这个字的的时候就会把这个 Cookie 保存到卡包里面，如果存在一个一摸一样的出入证（path、domain 都相同），则会进行覆盖。

每个属性的解释：

- path：设置 Cookie 的路径，如果不设置浏览器将其自动设置为当前请求的路径；
    - 例如请求的是 /login 页面，则自动设置为 /login;
- domain：设置 Cookie 的域名，如果不设置浏览器将其自动设置为当前请求的域名；
    - 如果服务器响应了一个无效的域，浏览器是不认的；
    - 比如浏览器请求的域是 zhangsan.tech，服务器响应头是`Set-cookie: a=1; domain=baidu.com`，这样的域浏览器是不认的；
- expire：设置 Cookie 的过期时间，必须是一个有效的 GMT 时间（格林威治时间），当客户端的时间达到设置的时间，则会自动销毁这个 Cookie；
- max-age：也是设置 Cookie 的过期时间，不过是一个相对的时间，表示多少秒之后 Cookie 就会过期；
    - 如果既没有设置 expire 也没有设置 max-age，则这个 Cookie 会在浏览器关闭后销毁；
- secure：表示是否是安全链接，如果设置了这个值，表示这个 Cookie 只能随着 HTTPS 协议发送到服务器；
- httponly：表示 Cookie 是否只用于网络传输，如果设置了这个值后本地的 JS 是无法获取这个 Cookie 的，这对跨站脚本攻击（XSS）很有用；

如果要删除一个 Cookie 只需要将 Cookie 的有效期设置一个过期的时间就可以了，例如：

```plain
set-cookie: token=; domain=zhangsan.tech; path=/; max-age=-1
```

## 🔢 客户端设置 Cookie
Cookie 存放在浏览器中，所以浏览器对 JS 脚本也开放了接口，可以设置 Cookie：

```js
document.cookie = "键=值; path=?; domain=?; expire=?; max-age=?; secure";
```

可以看到，JS 设置 Cookie 和响应头设置 Cookie 格式是一样的，只不过有下面的区别：

- 没有 httponly，因为该属性本来就是为了限制在客户端访问的，既然是在客户端配置，自然失去了限制的意义；

删除 Cookie 的话和服务器一样，只需要设置一个过期时间即可：

```js
document.cookie = "token=; domain=zhangsan.tech; path=/; max-age=-1";
```
