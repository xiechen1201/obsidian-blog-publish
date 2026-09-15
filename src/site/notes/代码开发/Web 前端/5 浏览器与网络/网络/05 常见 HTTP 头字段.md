---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/5 浏览器与网络/网络/05 常见 HTTP 头字段/","dg-note-properties":{}}
---

## Accept
Accept 存在于「请求头」中（客户端对服务端说的话），表示客户端希望接收到的数据类型。

Accept 的格式 type;q=value, type;q=value（这是两组信息，之间用逗号分隔）。

q 表示相对品质因子（权重），它从 0 到 1 的范围指定优先顺序（0 的优先级最低），没有指定质量值，默认为 q=1 ，如赋值为 0，则告诉服务器该内容类型不被浏览器接受！！！

下图为例，浏览器最希望接收到 text/html、application/xhtml+xml 其次是 application/xml ，再其次是其他任意数据类型，*_/*_ 表示任意类型。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1658731971754-6204128d-c26f-46f0-a0ec-5dbbbc2dac04.png)

Accept-Encoding 表示浏览器可以接受的压缩资源格式。

Accept-Language 表示返回资源的语言类型；例如：Accept-Language: zh-CN,en-US;q=0.8,en;q=0.6 就表示浏览器最希望接收到的语言是简体中文，其次是美国英语，再其次是其他形式的英语。

## User-Agent
该字段用于标识发起请求的客户端软件。它通常包含浏览器类型、版本、操作系统、设备信息和其他相关细节。

一个典型的 User-Agent 字符串通常包含以下部分：

- 浏览器信息：如 Chrome/85.0.4183.121；
- 操作系统：如 Windows NT 10.0；
- 设备信息：如移动设备可能包含设备型号；
- 其他信息：如语言设置或其他扩展信息；

## Content-Type
Content-Type 存在于「请求头」和「响应头」中，表示请求/返回资源类型和编码。

例如：Content-Type: text/html; charset=UTF-8；

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1681975585301-bab2c5d9-8015-45e6-9f90-264b8fe5b21f.png)

类似的还有其他一些子项：

- Content-Language：zh-CN ；表示返回资源的语言类型；
- Content-Encoding: gzip；表示服务器返回资源的编码格式（压缩格式，优化传输内容的大小）；

## Content-Length
Content-Length 用于描述 HTTP 消息实体的传输长度，存在于「请求头」和「响应头」中。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1682324180758-dcddd71b-db46-492e-93fd-e7298b9883cc.png)

例如当服务器返回 123456 的时候，Content-Length 就为 6；

- GET 请求：请求头没有 Content-Length，响应头带 Content-Length；
- POST 请求：请求头与响应头都带 Content-Length；

## Referer
Referer 表示来源域名，存在于「请求头」中。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1682324659445-b78e50db-5a1e-4c29-8b4f-bcf48d75d10d.png)

Referer 是一开始错误的写法，后来为了兼容也就保留了下来，referrer 才是正确写法，表示来源域名。

Referer 是请求头的一部分，当浏览器向 Web 服务器发送请求的时候，一般会带上 Referer，告诉服务器我是从哪个页面链接过来的。

我们可以在 HTML 代码中进行设置：

1、表示请求的时候不传递 referrer

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="referrer" content="no-referrer" />
    <title>Document</title>
  </head>
  <body>
    <a href="www.badu.com">点击访问</a>
  </body>
</html>
```

设置后访问 Baidu

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1682326589663-faaa8670-33a5-4e01-85c0-2918c6527d9a.png)

2、表示请求的时候 referrer 为 origin

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="referrer" content="origin" />
    <title>Document</title>
  </head>
  <body>
    <a href="www.badu.com">点击访问</a>
  </body>
</html>
```

设置后访问 Baidu

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1682326643001-230bbd39-bac9-473b-90c7-028fe72dbb67.png)

3、如果不进行设置，直接访问 Baidu

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/5%20%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8E%E7%BD%91%E7%BB%9C/%E7%BD%91%E7%BB%9C/_assets/1682326757311-2e968e60-0754-41d3-9fc1-4fd773a97f4c.png)

Referrer Policy: no-referrer-when-downgrade 仅当协议降级。例如 HTTPS 页面引入 HTTP 资源时不发送 Referrer 信息，这是大部分浏览器默认策略。

Referer 的应用场景：

- 收集用户访问页面的来源可以进去统计；
- 资源防止盗链（ 服务器拉取资源之前判断 referer 是否是自己的域名或 IP，如果不是就拦截，如果是则拉取资源）；
