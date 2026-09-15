---
{"dg-publish":true,"permalink":"//web/3/00-node/09-http/","dg-note-properties":{}}
---

## 🔢 发送请求
http 模块也可以用来请求一个地址：

```js
const http = require('http');
const request = http.request(url[, options][, callback]);
```

<br/>tips
参数：

- url：请求地址
- options：可选参数
    - headers：请求头
    - method：请求方法
    - ...
- callback：回调函数
    - 回调函数的参数为 http.IncomingMessage 实例对象。该实例对象可以拿到请求到的状态码和响应头信息，但是拿不到响应体的内容，因为服务返回的内容可能会很大，所以没有提供获取响应体的属性，需要使用流的方式读取！

返回值：

返回  http.ClientRequest 对象实例。

<br/>

案例：

```js
const http = require('http');

// 创建http请求
const request = http.request('http://www.baidu.com', { methods: 'GET' }, (res) => {
  // 返回 http.IncomingMessage 实例对象
  console.log(res);
  // 获取状态码
  console.log(res.statusCode);
  // 获取响应头
  console.log(res.headers);

  // http.IncomingMessage 继承 stream.Readable
  // 所以可以使用可读流
  res.on('data', (chunk) => {
    console.log('>>>>>>>获取内容：', chunk.toString());
  });

  res.on('end', () => {
    console.log('响应结束');
  });
});

// request 是 http.ClientRequest 实例对象
// 继承 Stream
// 写入一些东西
request.write('');
// 表示消息体完毕
request.end();
```

```latex
<ref *2> IncomingMessage {
  ...
}
200
{
  'accept-ranges': 'bytes',
  'cache-control': 'no-cache',
  connection: 'keep-alive',
  'content-length': '9508',
  'content-type': 'text/html',
  date: 'Mon, 03 Jun 2024 02:30:08 GMT',
  p3p: 'CP=" OTI DSP COR IVA OUR IND COM ", CP=" OTI DSP COR IVA OUR IND COM "',
  pragma: 'no-cache',
  server: 'BWS/1.1',
  'set-cookie': [
    'BAIDUID=9B06F19E9C56C6A4DDF9F2BC93028FD1:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com',
    'BIDUPSID=9B06F19E9C56C6A4DDF9F2BC93028FD1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com',
    'PSTM=1717381808; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com',
    'BAIDUID=9B06F19E9C56C6A42502AF4EA73CB91C:FG=1; max-age=31536000; expires=Tue, 03-Jun-25 02:30:08 GMT; domain=.baidu.com; path=/; version=1; comment=bd'
  ],
  traceid: '1717381808037795661812470185371405888991',
  vary: 'Accept-Encoding',
  'x-ua-compatible': 'IE=Edge,chrome=1',
  'x-xss-protection': '1;mode=block'
}
>>>>>>>获取内容： <!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"><meta content="always" name="referrer"><meta name="description" content="全球领先的中文搜索引擎、致力于让网民更便捷地获取信息，找到所求。百度超过千亿的中文网页数据库，可以瞬间找到相关的搜索结果。"><li
>>>>>>>获取内容： nk rel="shortcut icon" href="//www.baidu.com/favicon.ico" type="image/x-icon"><link rel="search" type="application/opensearchdescription+xml" href="//www.baidu.com/content-search.xml" title="百度搜索"><title>百度一下，你就知道</title><style type="text/css">body{margin:0;padding:0;text-align:center;background:#fff;height:100%}html{overflow-y:auto;color:#000;overflow:-moz-scrollbars;height:100%}body,input{font-size:12px;font-family:"PingFang SC",Arial,"Microsoft YaHei",sans-serif}a{text-decoration:none}a:hover{text-decoration:underline}img{border:0;-ms-interpolation-mode:bicubic}input{font-size:100%;border:0}body,form{position:relative;z-index:0}#wrapper{height:100%}#head_wrapper.s-ps-islite{padding-bottom:370px}#head_wrapper.s-ps-islite .s_form{position:relative;z-index:1}#head_wrapper.s-ps-islite .fm{position:absolute;bottom:0}#head_wrapper.s-ps-islite .s-p-top{position:absolute;bottom:40px;width:100%;height:181px}#head_wrapper.s-ps-islite #s_lg_img{position:static;margin:33px auto 0 auto;left:50%}#form{z-index:1}.s_form_wrapper{height:100%}#lh{margin:16px 0 5px;word-spacing:3px}.c-font-normal{font:13px/23px Arial,sans-serif}.c-color-t{color:#222}.c-btn,.c-btn:visited{color:#333!important}.c-btn{display:inline-block;overflow:hidden;font-family:inherit;font-weight:400;text-align:center;vertical-align:middle;outline:0;border:0;height:30px;width:80px;line-height:30px;font-size:13px;border-radius:6px;padding:0;background-color:#f5f5f6;cursor:pointer}.c-btn:hover{background-color:#315efb;color:#fff!important}a.c-btn{text-decoration:none}.c-btn-mini{height:24px;width:48px;line-height:24px}.c-btn-primary,.c-btn-primary:visited{color:#fff!important}.c-btn-primary{background-color:#4e6ef2}.c-btn-primary:hover{background-color:#315efb}a:active{color:#f60}#wrapper{position:relative;min-height:100%}#head{padding-bottom:100px;text-align:center}#wrapper{min-width:1250px;height:100%;min-height:600px}#head{position:relative;padding-bottom:0;height:100%;min-height:600px}.s_form_wrapper{height:100%}.quickdelete-wrap{position:relative}.tools{position:absolute;right:-75px}.s-isindex-wrap{position:relative}#head_wrapper.head_wrapper{width:auto}#head_wrapper{position:relative;height:40%;min-height:314px;max-height:510px;width:1000px;margin:0 auto}#head_wrapper .s-p-top{height:60%;min-height:185px;max-height:310px;position:relative;z-index:0;text-align:center}#head_wrapper input{outline:0;-webkit-appearance:none}#head_wrapper .s_btn_wr,#head_wrapper .s_ipt_wr{display:inline-block;zoom:1;background:0 0;vertical-align:top}#head_wrapper .s_ipt_wr{position:relative;width:546px}#head_wrapper .s_btn_wr{width:108px;height:44px;position:relative;z-index:2}#head_wrapper .s_ipt_wr:hover #kw{border-color:#a7aab5}#head_wrapper #kw{width:512px;height:16px;padding:12px 16px;font-size:16px;margin:0;vertical-align:top;outline:0;box-shadow:none;border-radius:10px 0 0 10px;border:2px solid #c4c7ce;background:#fff;color:#222;overflow:hidden;box-sizing:content-box}#head_wrapper #kw:focus{border-color:#4e6ef2!important;opacity:1}#head_wrapper .s_form{width:654px;height:100%;margin:0 auto;text-align:left;z-index:100}#head_wrapper .s_btn{cursor:pointer;width:108px;height:44px;line-height:45px;padding:0;background:0 0;background-color:#4e6ef2;border-radius:0 10px 10px 0;font-size:17px;color:#fff;box-shadow:none;font-weight:400;border:none;outline:0}#head_wrapper .s_btn:hover{background-color:#4662d9}#head_wrapper .s_btn:active{background-color:#4662d9}#head_wrapper .quickdelete-wrap{position:relative}#s_top_wrap{position:absolute;z-index:99;min-width:1000px;width:100%}.s-top-left{position:absolute;left:0;top:0;z-index:100;height:60px;padding-left:24px}.s-top-left .mnav{margin-right:31px;margin-top:19px;display:inline-block;position:relative}.s-top-left .mnav:hover .s-bri,.s-top-left a:hover{color:#315efb;text-decoration:none}.s-top-left .s-top-more-btn{padding-bottom:19px}.s-top-left .s-top-more-btn:hover .s-top-more{display:block}.s-top-right{position:absolute;right:0;top:0;z-index:100;height:60px;padding-right:24px}.s-top-right .s-top-right-text{margin-left:32px;margin-top:19px;display:inline-block;position:relative;vertical-align:top;cursor:pointer}.s-top-right .s-top-right-text:hover{color:#315efb}.s-top-right .s-top-login-btn{display:inline-block;margin-top:18px;margin-left:32px;font-size:13px}.s-top-right a:hover{text-decoration:none}#bottom_layer{width:100%;position:fixed;z-index:302;bottom:0;left:0;height:39px;padding-top:1px;overflow:hidden;zoom:1;margin:0;line-height:39px;background:#fff}#bottom_layer .lh{display:inline;margin-right:20px}#bottom_layer .lh:last-child{margin-left:-2px;margin-right:0}#bottom_layer .lh.activity{font-weight:700;text-decoration:underline}#bottom_layer a{font-size:12px;text-decoration:none}#bottom_layer .text-color{color:#bbb}#bottom_layer a:hover{color:#222}#bottom_layer .s-bottom-layer-content{text-align:center}</style></head><body><div id="wrapper" class="wrapper_new"><div id="head"><div id="s-top-left" class="s-top-left s-isindex-wrap"><a href="//news.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">新闻</a><a href="//www.hao123.com/" target="_blank" class="mnav c-font-normal c-color-t">hao123</a><a href="//map.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">地图</a><a href="//live.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">直播</a><a href="//haokan.baidu.com/?sfrom=baidu-top" target="_blank" class="mnav c-font-normal c-color-t">视频</a><a href="//tieba.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">贴吧</a><a href="//xueshu.baidu.com/" target="_blank" class="mnav c-font-normal c-color-t">学术</a><div class="mnav s-top-more-btn"><a href="//www.baidu.com/more/" name="tj_briicon" class="s-bri c-font-normal c-color-t" target="_blank">更多</a></div></div><div id="u1" class="s-top-right s-isindex-wrap"><a class="s-top-login-btn c-btn c-btn-primary c-btn-mini lb" style="position:relative;overflow:visible" name="tj_login" href="//www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1">登录</a></div><div id="head_wrapper" class="head_wrapper s-isindex-wrap s-ps-islite"><div class="s_form"><div class="s_form_wrapper"><div id="lg" class="s-p-top"><img hidefocus="true" id="s_lg_img" class="index-logo-src" src="//www.baidu.com/img/flexible/logo/pc/index.png" width="270" height="129" usemap="#mp"><map name="mp"><area style="outline:0" hidefocus="true" shape="rect" coords="0,0,270,129" href="//www.baidu.com/s?wd=%E7%99%BE%E5%BA%A6%E7%83%AD%E6%90%9C&amp;sa=ire_dl_gh_logo_texing&amp;rsv_dl=igh_logo_pcs" target="_blank" title="点击一下，��解更多"></map></div><a href="//www.baidu.com/" id="result_logo"></a><form id="form" name="f" action="//www.baidu.com/s" class="fm"><input type="hidden" name="ie" value="utf-8"> <input type="hidden" name="f" value="8"> <input type="hidden" name="rsv_bp" value="1"> <input type="hidden" name="rsv_idx" value="1"> <input type="hidden" name="ch" value=""> <input type="hidden" name="tn" value="baidu"> <input type="hidden" name="bar" value=""> <span class="s_ipt_wr quickdelete-wrap"><input id="kw" name="wd" class="s_ipt" value="" maxlength="255" autocomplete="off"> </span><span class="s_btn_wr"><input type="submit" id="su" value="百度一下" class="bg s_btn"> </span><input type="hidden" name="rn" value=""> <input type="hidden" name="fenlei" value="256"> <input type="hidden" name="oq" value=""> <input type="hidden" name="rsv_pq" value="b9ff093e0000e419"> <input type="hidden" name="rsv_t" value="3635FYbdbC8tlWmudZmYaUnaucNe+RzTzNEGqg/JuniQU10WL5mtMQehIrU"> <input type="hidden" name="rqlang" value="cn"> <input type="hidden" name="rsv_enter" value="1"> <input type="hidden" name="rsv_dl" value="ib"></form></div></div></div><div id="bottom_layer" class="s-bottom-layer s-isindex-wrap"><div class="s-bottom-layer-content"><p class="lh"><a class="text-color" href="//home.baidu.com/" target="_blank">关于百度</a></p><p class="lh"><a class="text-color" href="//ir.baidu.com/" target="_blank">About Baidu</a></p><p class="lh"><a class="text-color" href="//www.baidu.com/duty" target="_blank">使用百度前必读</a></p><p class="lh"><a class="text-color" href="//help.baidu.com/" target="_blank">帮助中心</a></p><p class="lh"><a class="text-color" href="//www.beian.gov.cn/portal/registerSystemInfo?recordcode=11000002000001" target="_blank">京公网安备11000002000001号</a></p><p class="lh"><a class="text-color" href="//beian.miit.gov.cn/" target="_blank">京ICP证030173号</a></p><p class="lh"><span id="year" class="text-color"></span></p><p class="lh"><span class="text-color">互联网药品信息服务资格证书 (京)-经营性-2017-0020</span></p><p class="lh"><a class="text-color" href="//www.baidu.com/licence/" target="_blank">信息网络传播视听节目许可证 0110516</a></p></div></div></div></div><script type="text/javascript">var date=new Date,year=date.getFullYear();document.getElementById("year").innerText="©"+year+" Baidu "</script></body></html>
响应结束
```

以上是一个 GET 的请求示例，POST 请求和 GET 基本一致，只是 POST 需要调用`request.write()`来写入传递的参数：

```js
const request = http.request('http://www.baidu.com', {
  methods: 'POST'
}, (res) => {
  // do...
});
request.write('name=zhangsan&age=20');
request.end();
```

## 🔢 创建请求
```js
const http = require('http');

const server = http.createServer([options][, requestListener]);
```

<br/>tips
参数：

- options：可选的参数
    - ...
- requestListener：请求监听函数，函数会得到两个参数 ：
    - http.IncomingMessage 可以获取客户端请求的信息
    - http.ServerResponse 写入信息可以返回到客户端

返回：

返回 http.Server 实例对象，继承 net.Server 类。

<br/>

示例：

```js
const http = require('http');
const url = require('url');

// 创建一个服务器
const server = http.createServer({}, (req, res) => {
  // 请求内容
  console.log('有请求来了');
  console.log('请求地址', req.url);
  console.log(url.parse(req.url));
  req.on('data', (chunk) => {
    console.log('>>>>>>>获取内容：', chunk.toString());
  });

  // 响应内容
  res.setHeader('Content-Type', 'text/html;charset=utf-8');
  res.write('<h1>hello world</h1>');
  res.end();
});

// 创建端口
server.listen(9527);

server.on('listening', function () {
  console.log('服务器启动成功，端口 9527');
});
```

这样当客户端访问 http://localhost:9527 的时候就能看到我们返回的内容：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1717383118681-f39962b8-57ed-4675-9f51-2f3747219660.png)

## 🔢 案例：创建静态资源服务器
目录结构：

```latex
.
├── server.js
└── static
    ├── index.css
    ├── index.html
    ├── index.jpg
    └── index.js
```

static 目录下存放的是前端需要的文件。

```js
const http = require('http');
const URL = require('url');
const path = require('path');
const fs = require('fs/promises');

// 获取文件信息
async function getState(filename) {
  try {
    return await fs.stat(filename);
  } catch (error) {
    return null;
  }
}

// 获取要处理的文件信息
async function getFileInfo(url) {
  const urlObj = URL.parse(url); // 使用 url 模块解析路径
  let filename;
  // 因为请求的地址是 / 开头的，所以需要把 / 截取掉，否则就是绝对路径
  filename = path.resolve(__dirname, './static', urlObj.pathname.substring(1));
  // 获取文件信息
  let state = await getState(filename);

  // 如果文件不存在
  if (!state) {
    // 文件不存在
    return null;
  } else if (state.isDirectory()) {
    // 如果文件是一个目录
    // 那就拼接上 index.html 再次进行查找
    filename = path.resolve(
      __dirname,
      './static',
      urlObj.pathname.substring(1),
      'index.html'
    );
    state = getState(filename);

    if (!state) {
      // 目录下没有 index.html
      return null;
    } else {
      // 找到 index.html
      return await fs.readFile(filename);
    }
  } else {
    // 如果找到文件
    return await fs.readFile(filename);
  }
}

// 请求/响应处理函数
async function handler(req, res) {
  const info = await getFileInfo(req.url);
  if (info) {
    // 如果文件存在就通过流的方式写入文件返回
    res.write(info);
    res.end();
  } else {
    // 否则就返回404
    res.statusCode = 404;
    res.write('<h1>Not Fount</h1>');
    res.end();
  }
}

const server = http.createServer(handler);
server.listen(3000);
server.on('listening', () => {
  console.log('server is running at http://localhost:3000');
});

```

客户端访问 http://localhost:3000 响应正常：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1717384039839-4c700ea5-7faf-4bed-9cf0-fe46e4ee2a7a.png)
