---
{"dg-publish":true,"permalink":"//web/3/00-node/08-net/","dg-note-properties":{}}
---

## 🔢 回顾 http 的请求
在 HTTP 请求中分为两种模式：

- 普通模式
    - ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716880597691-1b667ff5-6d38-458d-a892-2df8751ae813.png)
- 长链接模式
    - 请求页面的时候会产生很多的请求，例如图片、JS 文件等等，如果使用普通模式就会造成响应时间的降低，长链接模式可以在一个非常短的时间内共用一个 TCP 请求。
    - ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716880603344-fb487a7c-1645-4f5f-a67a-badac29e2511.png)
    - 例如在浏览器中发起一个请求，`keep-alive`表示告诉服务器不要那么着急的关掉 TCP 服务，后续可能还有别的请求。
        * ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716881469076-a15e956d-1391-4713-898c-7eee6cf3c0d3.png)

## 🔢 net 模块能干什么
net 模块是一个通信模块，可以传输数据。利用它可以实现：

- 进程间的通信 IPC；
- 网络通信 TCP/IP；
- 提供了对底层网络通信的直接访问；

## 🔢 创建客户端
使用`createConnection()`方法创建一个客户端链接：

```js
const net = require("net");

net.createConnection(options[, connectListener])
```

参数：

- options：选项，常见的属性有：
    - host：主机名；
    - port：端口号；
- connectListener：可选的回调函数，当连接成功建立时会被调用；

<br/>

返回值：

返回一个 socket 对象，用于启动连接的新创建的套接字。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716882571492-3e146673-5e92-45d3-97b0-76287976c740.png)

socket 是一个特殊的文件，在 Node 中表现为一个双工流（可读可写）的对象，通过向流写入内容发送数据，通过监听流的内容获取数据！

<br/>

例如使用 net 模块请求一个地址并获取响应的数据：

```js
const net = require('net');

// 创建客户端
const socket = net.createConnection(
  {
    // 设置请求主机
    host: 'www.baidu.com',
    port: 80
  },
  () => {
    console.log('链接成功');
  }
);

// 写入数据（发送请求）
socket.write(`GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive

`);

// 监听响应
socket.on('data', (chunk) => {
  const response = chunk.toString('utf-8');
  console.log(">>>>>>>读取到数据", response);

  // 手动关闭链接
  // socket.end();
});

socket.on('close', () => {
  console.log('链接关闭');
});
```

以下是请求到的内容：

```latex
链接成功
>>>>>>>读取到数据 HTTP/1.1 200 OK
Accept-Ranges: bytes
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 9508
Content-Type: text/html
Date: Tue, 28 May 2024 08:11:03 GMT
P3p: CP=" OTI DSP COR IVA OUR IND COM "
P3p: CP=" OTI DSP COR IVA OUR IND COM "
Pragma: no-cache
Server: BWS/1.1
Set-Cookie: BAIDUID=03186AD1F9FDE832EFE3BEB9AFDDCF09:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: BIDUPSID=03186AD1F9FDE832EFE3BEB9AFDDCF09; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: PSTM=1716883863; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: BAIDUID=03186AD1F9FDE832C95549EDFCD5A583:FG=1; max-age=31536000; expires=Wed, 28-May-25 08:11:03 GMT; domain=.baidu.com; path=/; version=1; comment=bd
Traceid: 1716883863026064717811639852713549879064
Vary: Accept-Encoding
X-Ua-Compatible: IE=Edge,chrome=1
X-Xss-Protection: 1;mode=block

<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"><meta content="always" name="referrer"><meta name="description" content="全球领先的中文搜索引擎、致力于让网民更便捷地获取信息，找到所求。百度超过千亿的中文网页数据库，可以瞬间找到相关的搜索结果。"><link rel="shortcut icon" href="//www.baidu.com/favicon.ico" type="image/x-icon"><link rel="search
>>>>>>>读取到数据 " type="application/opensearchdescription+xml" href="//www.baidu.com/content-search.xml" title="百度搜索"><title>百度一下，你就知道</title><style type="text/css">body{margin:0;padding:0;text-align:center;background:#fff;height:100%}html{overflow-y:auto;color:#000;overflow:-moz-scrollbars;height:100%}body,input{font-size:12px;font-family:"PingFang SC",Arial,"Microsoft YaHei",sans-serif}a{text-decoration:none}a:hover{text-decoration:underline}img{border:0;-ms-interpolation-mode:bicubic}input{font-size:100%;border:0}body,form{position:relative;z-index:0}#wrapper{height:100%}#head_wrapper.s-ps-islite{padding-bottom:370px}#head_wrapper.s-ps-islite .s_form{position:relative;z-index:1}#head_wrapper.s-ps-islite .fm{position:absolute;bottom:0}#head_wrapper.s-ps-islite .s-p-top{position:absolute;bottom:40px;width:100%;height:181px}#head_wrapper.s-ps-islite #s_lg_img{position:static;margin:33px auto 0 auto;left:50%}#form{z-index:1}.s_form_wrapper{height:100%}#lh{margin:16px 0 5px;word-spacing:3px}.c-font-normal{font:13px/23px Arial,sans-serif}.c-color-t{color:#222}.c-btn,.c-btn:visited{color:#333!important}.c-btn{display:inline-block;overflow:hidden;font-family:inherit;font-weight:400;text-align:center;vertical-align:middle;outline:0;border:0;height:30px;width:80px;line-height:30px;font-size:13px;border-radius:6px;padding:0;background-color:#f5f5f6;cursor:pointer}.c-btn:hover{background-color:#315efb;color:#fff!important}a.c-btn{text-decoration:none}.c-btn-mini{height:24px;width:48px;line-height:24px}.c-btn-primary,.c-btn-primary:visited{color:#fff!important}.c-btn-primary{background-color:#4e6ef2}.c-btn-primary:hover{background-color:#315efb}a:active{color:#f60}#wrapper{position:relative;min-height:100%}#head{padding-bottom:100px;text-align:center}#wrapper{min-width:1250px;height:100%;min-height:600px}#head{position:relative;padding-bottom:0;height:100%;min-height:600px}.s_form_wrapper{height:100%}.quickdelete-wrap{position:relative}.tools{position:absolute;right
>>>>>>>读取到数据 :-75px}.s-isindex-wrap{position:relative}#head_wrapper.head_wrapper{width:auto}#head_wrapper{position:relative;height:40%;min-height:314px;max-height:510px;width:1000px;margin:0 auto}#head_wrapper .s-p-top{height:60%;min-height:185px;max-height:310px;position:relative;z-index:0;text-align:center}#head_wrapper input{outline:0;-webkit-appearance:none}#head_wrapper .s_btn_wr,#head_wrapper .s_ipt_wr{display:inline-block;zoom:1;background:0 0;vertical-align:top}#head_wrapper .s_ipt_wr{position:relative;width:546px}#head_wrapper .s_btn_wr{width:108px;height:44px;position:relative;z-index:2}#head_wrapper .s_ipt_wr:hover #kw{border-color:#a7aab5}#head_wrapper #kw{width:512px;height:16px;padding:12px 16px;font-size:16px;margin:0;vertical-align:top;outline:0;box-shadow:none;border-radius:10px 0 0 10px;border:2px solid #c4c7ce;background:#fff;color:#222;overflow:hidden;box-sizing:content-box}#head_wrapper #kw:focus{border-color:#4e6ef2!important;opacity:1}#head_wrapper .s_form{width:654px;height:100%;margin:0 auto;text-align:left;z-index:100}#head_wrapper .s_btn{cursor:pointer;width:108px;height:44px;line-height:45px;padding:0;background:0 0;background-color:#4e6ef2;border-radius:0 10px 10px 0;font-size:17px;color:#fff;box-shadow:none;font-weight:400;border:none;outline:0}#head_wrapper .s_btn:hover{background-color:#4662d9}#head_wrapper .s_btn:active{background-color:#4662d9}#head_wrapper .quickdelete-wrap{position:relative}#s_top_wrap{position:absolute;z-index:99;min-width:1000px;width:100%}.s-top-left{position:absolute;left:0;top:0;z-index:100;height:60px;padding-left:24px}.s-top-left .mnav{margin-right:31px;margin-top:19px;display:inline-block;position:relative}.s-top-left .mnav:hover .s-bri,.s-top-left a:hover{color:#315efb;text-decoration:none}.s-top-left .s-top-more-btn{padding-bottom:19px}.s-top-left .s-top-more-btn:hover .s-top-more{display:block}.s-top-right{position:absolute;right:0;top:0;z-index:100;height:60px;padding-right:24px}.s-top-right .s-top-right-text{margin-left:32px;margin-top:19px;display:inline-block;position:relative;vertical-align:top;cursor:pointer}.s-top-right .s-top-right-text:hover{color:#315efb}.s-top-right .s-top-login-btn{display:inline-block;margin-top:18px;margin-left:32px;font-size:13px}.s-top-right a:hover{text-decoration:none}#bottom_layer{width:100%;position:fixed;z-index:302;bottom:0;left:0;height:39px;padding-top:1px;overflow:hidden;zoom:1;margin:0;line-height:39px;background:#fff}#bottom_layer .lh{display:inline;margin-right:20px}#bottom_layer .lh:last-child{margin-left:-2px;margin-right:0}#bottom_layer .lh.activity{font-weight:700;text-decoration:underline}#bottom_layer a{font-size:12px;text-decoration:none}#bottom_layer .text-color{color:#bbb}#bottom_layer a:hover{color:#222}#bottom_layer .s-bottom-layer-content{text-align:center}</style></head><body><div id="wrapper" class="wrapper_new"><div id="head"><div id="s-top-left" class="s-top-left s-isindex-wrap"><a href="//news.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">新闻</a><a href="//www.hao123.com/" target="_blank" class="mnav c-font-normal c-color-t">hao123</a><a href="//map.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">地图</a><a href="//live.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">直播</a><a href="//haokan.baidu.com/?sfrom=baidu-top" target="_blank" class="mnav c-font-normal c-color-t">视频</a><a href="//tieba.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">贴吧</a><a href="//xueshu.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">学术</a><div class="mnav s-top-more-btn"><a href="//www.baidu.com/more/" name="tj_briicon" class="s-bri c-font-normal c-color-t" target="_blank">更多</a></div></div><div id="u1" class="s-top-right s-isindex-wrap"><a class="s-top-login-btn c-btn c-btn-primary c-btn-mini lb" style="position:relative;overflow:visible" name="tj_login" href="//www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1">登录</a></div><div id="head_wrapper" class="head_wrapper s-isindex-wrap s-ps-islite"><div class="s_form"><div class="s_form_wrapper"><div id="lg" class="s-p-top"><img hidefocus="true" id="s_lg_img" class="index-logo-src" src="//www.baidu.com/img/flexible/logo/pc/index.png" width="270" height="129" usemap="#mp"><map name="mp"><area style="outline:0" hidefocus="true" shape="rect" coords="0,0,270,129" href="//www.baidu.com/s?wd=%E7%99%BE%E5%BA%A6%E7%83%AD%E6%90%9C&amp;sa=ire_dl_gh_logo_texing&amp;rsv_dl=igh_logo_pcs" target="_blank" title="点击一下，了解更多"></map></div><a href="//www.baidu.com/" id="result_logo"></a><form id="form" name="f" action="//www.baidu.com/s" class="fm"><input type="hidden" name="ie" value="utf-8"> <input type="hidden" name="f" value="8"> <input type="hidden" name="rsv_bp" value="1"> <input type="hidden" name="rsv_idx" value="1"> <input type="hidden" name="ch" value=""> <input type="hidden" name="tn" value="baidu"> <input type="hidden" name="bar" value=""> <span class="s_ipt_wr quickdelete-wrap"><input id="kw" name="wd" class="s_ipt" value="" maxlength="255" autocomplete="off"> </span><span class="s_btn_wr"><input type="submit" id="su" value="百度一下" class="bg s_btn"> </span><input type="hidden" name="rn" value=""> <input type="hidden" name="fenlei" value="256"> <input type="hidden" name="oq" value=""> <input type="hidden" name="rsv_pq" value="b9ff093e0000e419"> <input type="hidden" name="rsv_t" value="3635FYbdbC8tlWmudZmYaUnaucNe+RzTzNEGqg/JuniQU10WL5mtMQehIrU"> <input type="hidden" name="rqlang" value="cn"> <input type="hidden" name="rsv_enter" value="1"> <input type="hidden" name="rsv_dl" value="ib"></form></div></div></div><div id="bottom_layer" class="s-bottom-layer s-isindex-wrap"><div class="s-bottom-layer-content"><p class="lh"><a class="text-color" href="//home.baidu.com/" target="_blank">关于百度</a></p><p class="lh"><a class="text-color" href="//ir.baidu.com/" target="_blank">About Baidu</a></p><p class="lh"><a class="text-color" href="//www.baidu.com/duty" target="_blank">使用百度前必读</a></p><p class="lh"><a class="text-color" href="//help.baidu.com/" target="_blank">帮助中心</a></p><p class="lh"><a class="text-color" href="//www.beian.gov.cn/portal/registerSystemInfo?recordcode=11000002000001" target="_blank">京公网安备11000002000001号</a></p><p class="lh"><a class="text-color" href="//beian.miit.gov.cn/" target="_blank">京ICP证030173号</a></p><p class="lh"><span id="year" class="text-color"></span></p><p class="lh"><span class="text-color">互联网药品信息服务资格证书 (京)-经营性-2017-0020</span></p><p class="lh"><a class="text-color" href="//www.baidu.com/licence/" target="_blank">信息网络传播视听节目许可证 0110516</a></p></div></div></div></div><script type="text/javascript">var date=new Date,year=date.getFullYear();document.getElementById("year").innerText="©"+year+" Baidu "</script></body></html>
链接关闭
```

## 🔢 创建服务器
使用`createServer()`方法创建一个服务器：

```js
const net = require('net');

const server = net.createServer([options][, connectionListener])
```

参数：

- options: 选项；
- 可选的回调函数，当连接成功建立时会被调用；

<br/>

返回值：

返回一个 server 对象。

<br/>

方法与事件：

```js
// 监听当前计算机中的某个端口
server.listen(port);
// 开始监听端口后触发的事件
server.on('listening', ()=>{})

// 当某个链接到来时触发的事件
// 触发事件侯会返回一个 socket 对象
server.on('connection', (socket)=>{
  console.log(socket);
})
```

例如使用 net 模块创建一个服务器并监听：

```js
const net = require('net');

const server = net.createServer();

server.listen(9527);

server.on('listening', () => {
    console.log('server is listening on port 9527');
});

// 有客户端已经链接到服务器
server.on('connection', (socket) => {
    console.log('有客户端链接到服务器~');
    console.log(socket);

    // 读取内容（收到响应）
    socket.on('data', chunk) => {
        console.log('客户端发送的数据是：', chunk.toString());
        // 写入内容（返回响应）
        socket.write('您好！');
    });

    // 关闭
    socket.on('close', () => {
        console.log('链接关闭了');
    });
});
```

这样当浏览器访问`http://localhost:9527`的时候，node 就会监听：

```plain
server is listening on port 9527
有客户端链接到服务器~
<ref *2> Socket {
  connecting: false,
  _hadError: false,
  _parent: null,
  _host: null,
  _closeAfterHandlingError: false,
  _events: {
    ...
  },
  _readableState: ReadableState {
    ...
  },
  _writableState: WritableState {
    ...
  },
  allowHalfOpen: false,
  _maxListeners: undefined,
  _eventsCount: 1,
  _sockname: null,
  _pendingData: null,
  _pendingEncoding: '',
  server: <ref *1> Server {
    ...
  },
  _server: <ref *1> Server {
    ...
  },
  [Symbol(async_id_symbol)]: 7,
  [Symbol(kHandle)]: TCP {
    ...
  },
  [Symbol(lastWriteQueueSize)]: 0,
  [Symbol(timeout)]: null,
  [Symbol(kBuffer)]: null,
  [Symbol(kBufferCb)]: null,
  [Symbol(kBufferGen)]: null,
  [Symbol(shapeMode)]: true,
  [Symbol(kCapture)]: false,
  [Symbol(kSetNoDelay)]: false,
  [Symbol(kSetKeepAlive)]: false,
  [Symbol(kSetKeepAliveInitialDelay)]: 0,
  [Symbol(kBytesRead)]: 0,
  [Symbol(kBytesWritten)]: 0
}
有客户端链接到服务器~
<ref *2> Socket {
  connecting: false,
  _hadError: false,
  _parent: null,
  _host: null,
  _closeAfterHandlingError: false,
  _events: {
    ...
  },
  _readableState: ReadableState {
    ...
  },
  _writableState: WritableState {
    ...
  },
  allowHalfOpen: false,
  _maxListeners: undefined,
  _eventsCount: 1,
  _sockname: null,
  _pendingData: null,
  _pendingEncoding: '',
  server: <ref *1> Server {
    ...
  },
  _server: <ref *1> Server {
    ...
  },
  [Symbol(async_id_symbol)]: 10,
  [Symbol(kHandle)]: TCP {
    ...
  },
  [Symbol(lastWriteQueueSize)]: 0,
  [Symbol(timeout)]: null,
  [Symbol(kBuffer)]: null,
  [Symbol(kBufferCb)]: null,
  [Symbol(kBufferGen)]: null,
  [Symbol(shapeMode)]: true,
  [Symbol(kCapture)]: false,
  [Symbol(kSetNoDelay)]: false,
  [Symbol(kSetKeepAlive)]: false,
  [Symbol(kSetKeepAliveInitialDelay)]: 0,
  [Symbol(kBytesRead)]: 0,
  [Symbol(kBytesWritten)]: 0
}
客户端发送的数据是： GET / HTTP/1.1
Host: localhost:9527
Connection: keep-alive
Cache-Control: max-age=0
sec-ch-ua: "Google Chrome";v="125", "Chromium";v="125", "Not.A/Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9
Cookie: sidebarStatus=1; Authorization=bearer%20eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsaWNlbnNlIjoibWFkZSBieSB3YW5neSIsInVzZXJfbmFtZSI6IjE3MDkyMTY3OTg4Iiwic2NvcGUiOlsic2VydmVyIl0sImV4cCI6MTcxNjk2MTczNywidXNlcklkIjoyNjYwMDAzLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIiwiUk9MRV9VU0VSIl0sImp0aSI6ImUwY2FjYTkzLWM0ODgtNDI2NC05NzYyLTlkNDgwZTA5MDI4ZiIsImNsaWVudF9pZCI6InBpZyJ9.zSxcaxTX5mG3O1wihAwlNBQIQGBt5SbWewKtDIVPhBY

链接关闭了
```

<br/>warning
⚠️ 注意

从上面的打印结果来看，发现有两次「有客户端链接到服务器~」的打印是为什么？

这是因为有一次是测试请求。

<br/>

使用 net 监听响应并返回数据内容：

```js
const net = require('net');
const fs = require('fs/promises');
const path = require('path');

const server = net.createServer();

server.listen(9527);

server.on('listening', () => {
  console.log('server is listening on port 9527');
});

// 有客户端已经链接到服务器
server.on('connection', (socket) => {
  socket.on('data', async (chunk) => {
    // 读取文件
    const bodyBuffer = await fs.readFile(
      path.resolve(__dirname, '../note/image.png')
    );
    const headBuffer = Buffer.from(
      `HTTP/1.1 200 OK
Content-Type: image/jpeg

`,
      'utf-8'
    );
    // 合并内容
    const result = Buffer.concat([headBuffer, bodyBuffer]);
    // 响应内容
    socket.write(result);
    // 结束
    socket.end();
  });
});
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1716886525244-c32894b5-b85d-462c-9972-fb5884ea1e57.png)
