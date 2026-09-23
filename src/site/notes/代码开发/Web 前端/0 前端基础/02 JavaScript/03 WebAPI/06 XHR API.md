---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/06 XHR API/","dg-note-properties":{}}
---

## 前言
之前我们学习过 HTTP 协议，它是浏览器和服务器之间通信的协议。

在 AJAX 出现之前都是用标签的资源来发起 HTTP 请求，比如`img/src`、`a/href`、`script/src`。

在 AJAX 之前想要请求一段数据，需要进行混编模式开发，但是这样每次点击按钮请求数据的数据都会造成页面的刷新（`<a>`标签 > 点击 > 当前页面 URL 带参数 > 页面刷新后判断 URL 是否存在参数，如果有参数加载数据）。

> 混编模式：前端和后端的代码写在一起，文件拓展名是后端语言的拓展名（index.php），因为浏览器不能解析后端语言代码，但是 php 文件可以嵌入 HTML 代码。
>

以上就是原始开发网站的模式。

那么如何做到不重新加载整个页面，却能获取到新的网页所需的数据和更新部分网页内容呢？

## 认识 AJAX
AJAX 的全称是 Asynchronous JavaScript and XML，意思是异步的 JavaScript 和 XML。

有了 AJAX 后可以利用 JavaScript 脚本直接发起 HTTP 请求。

当请求服务器返回 JSON/XML 文档，前端从 XML 文档中提取数据，在不刷新整个网页的基础上，渲染到网页相应的位置。

AJAX 在 1999 年之前都是通过 HTML 的资源发起 HTTP 请求，而 IE5.0 允许了允许 JS 脚本发起 HTTP 请求（异步）。到了 2005 谷歌地图使用异步技术更新地图服务这才得到了诸多大厂的青睐，到了 2006 年 W3C 发布了 AJAX 国际标准。

## 使用 Ajax
使用 AJAX 先要创建`XMLHttpRequest`实例对象和`ActiveXObject`实例对象（IE5 和 IE6 专用）。

`XMLHttpRequest`是浏览器内置的构造函数，需要进行实例化，例如`new Object()`、`new Date()`、`new Regexp()`等都一样。

> [!tip]
>
> Info
>
> `XMLHttpRequest`的名字中为什么包含 XML 呢？
>
> 因为当时异步请求只支持 XML，现在我们通常请求的是多种资源，故这个名字已经不准确了，这个名字只是延用。
>
> AJAX 请求 XML 并解析的示例：
>
> ![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1742351992974-540e9f60-3f8e-4ef1-9dc2-60925cf672bf.png)


## 创建实例
创建 AJAX 实例对象：

```js
// 兼容写法
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}
```

## open 方法
使用`xhr`对象首先要调用`open()`方法，这个方法接收 3 个参数：请求类型、请求 URL，以及表示请求是否异步的布尔值(`true`异步/`false`同步)。

调用`open()`不会实际发送请求，只是为发送请求做好准备。

```js
// 兼容写法
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

xhr.open("GET","/user/info",true)
```

## send() 方法
要发送定义好的请求，必须调用`send()`方法，`send()`方法接收一个参数，是作为请求体发送的数据。

如果不需要发送请求体，则必须传`null`，因为这个参数在某些浏览器中是必需的。

```js
// 兼容写法
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

xhr.open("GET","/user/info",true);
xhr.send(null);
```

## HTTP 头部
默认情况下，`xhr`请求会发送相关的头信息，如果需要发送额外的请求头部，可以使用`setRequestHeader()`方法。

这个方法接收两个参数：头部字段的名称和值。为保证请求头部被发送，必须在`open()`之后、`send()`之前调用`setRequestHeader()`。

```js
// 兼容 IE 低版本
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

xhr.open("GET", "/user/info", true);
// 设置请求头
xhr.setRequestHeader("Content-Type", "application/json"); // 指定请求的数据类型
xhr.setRequestHeader("Authorization", "Bearer your_token_here"); // 例如添加身份验证
xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest"); // 让服务器识别这是 AJAX 请求
xhr.send(null);
```

## readyState 属性与 readystatechange 事件
`xhr`对象有一个`readyState`属性，表示当前处在请求/响应过程的哪个阶段。

- 0: 请求未初始化；
- 1: 服务器连接已建立；
- 2: 请求已接收；
- 3: 请求处理中；
- 4: 请求已完成，且响应已就绪；

> [!warning]
>
> `readyState`仅仅是针对请求的状态码，获取资源是否成功取决于`status`的状态。


每次`readyState`从一个值变成另一个值，都会触发`readystatechange`事件。

为保证跨浏览器兼容，`onreadystatechange`事件处理程序应该在调用`open()`之前赋值。

```js
// 兼容 IE 低版本
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

// 监听 readyState 变化
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if (xhr.status >= 200 && xhr.status < 300) {
      console.log("请求成功:", xhr.responseText);
    } else {
      console.error("请求失败，状态码:", xhr.status);
    }
  }
};

xhr.open("GET", "/user/info", true);
xhr.setRequestHeader("Content-Type", "application/json");
xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
xhr.send(null);
```

## 响应
收到响应后，`xhr`对象的以下属性会被填充上数据：

- `responseText`：作为响应体返回的文本；
- `responseXML`：如果响应的内容类型是 text/xml 或 application/xml，那就是包含响应数据的XML DOM 文档；
- `status`：响应的 HTTP 状态；
- `statusText`：响应的 HTTP 状态描述；

收到响应后，第一步要检查`status`属性以确保响应成功返回。

一般来说，HTTP 状态码为 2xx 表示成功。此时，`responseText`或 `responseXML`（如果内容类型正确）属性中会有内容。

如果`HTTP`状态码是 304，则表示资源未修改过，是从浏览器缓存中直接拿取的。当然这也意味着响应有效。为确保收到正确的响应，应该检查这些状态。

## 案例
```js
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status < 304) {
      var resData = JSON.parse(xhr.responseText);
      console.log("success", resData);
    }
  }
};

xhr.open("GET", "/user/info?id=123456", true);
xhr.send();
```

```js
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status < 304) {
      var resData = JSON.parse(xhr.responseText);
      console.log("success", resData);
    }
  }
};

xhr.open("POST", "/user/info", true);
// POST 是以表单方式提交，所以需要设置请求头，setRequestHeader 必须在 send 之前设置
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
// POST 参数必须进行序列化，目的是请求体中的数据转换为键值对，这是请求报文真正的样子，浏览器只是进行了美化
// 后端接收到 a=1&b=2&c=3 这样的数据才知道是这是一个 POST 方式传来的数据
xhr.send("a=1&b=2&c=3");
```

## 封装 AJAX
接下来我们封装一下 AJAX (简单版)：

```js
// 立即执行函数，返回一个对象
var $ = (function () {
  // 兼容写法
  var xhr = window.XMLHttpRequest
  ? new XMLHttpRequest()
  : new ActiveXObject("Microsoft.XMLHTTP");

  if (!xhr) {
    throw new Error("浏览器不支持异步发起 HTTP 请求！");
  }

  // 对象序列化
  function formatData(object) {
    var str = "";
    for (const key in object) {
      str += key + "=" + object[key] + "&";
    }
    return str.replace(/&$/, "");
  }

  // 处理 Ajax 请求
  function _doAjax(opt) {
    var opt = opt || {},
        type = (opt.type || "GET").toUpperCase(),
        url = opt.url,
        async = opt.async || true,
        data = opt.data || null,
        success = opt.success || function () {},
        error = opt.error || function () {},
        complete = opt.complete || function () {};

    if (!url) {
      throw new Error("url 不能为空！");
    }

    xhr.open(type, url, async);
    if (type === "POST") {
      xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    }
    if (type === "GET") {
      xhr.send();
    } else {
      var str = formatData(data);
      xhr.send(str);
    }

    xhr.onreadystatechange = function () {
      if (xhr.readyState === 4 && xhr.status === 200) {
        success(JSON.parse(xhr.responseText));
        complete();
      } else {
        error();
        complete();
      }
    };
  }

  return {
    ajax: function (opt) {
      _doAjax(opt);
    },
    get: function (url, successCallback, errorCallback) {
      _doAjax({
        url,
        type: "GET",
        success: successCallback,
        error: errorCallback,
      });
    },
    post: function (url, data, successCallback, errorCallback) {
      _doAjax({
        url,
        type: "POST",
        data,
        success: successCallback,
        error: errorCallback,
      });
    },
  };
})();

// 调用 ajax
$.ajax({
  url: "user/list",
  type: "POST",
  data: {
    a: 1,
    b: 2,
  },
  success: function (res) {
    console.log("success");
  },
  error: function (res) {
    console.log("error");
  },
});

// 调用 GET 方法
$.get(
  "user/detail",
  function (res) {
    console.log("success");
  },
  function (res) {
    console.log("error");
  }
);
// 调用 POST 方法
$.post(
  "/user/edit",
  {
    a: 1,
    b: 2,
  },
  function (res) {
    console.log("success");
  },
  function (res) {
    console.log("error");
  }
);
```

## XMLHttpRequest Level2
`XMLHttpRequest`标准又分为 Level 1和 Level 2 （2012 年发布）两个版本。

`XMLHttpRequest`Level 1 缺点：

- 无法发送跨域请求；
- 不能非纯文本的数据；
- 无法获取传输进度；

`XMLHttpRequest`Level 2 改进：

- 可以发送跨域请求；
- 支持获取二进制数据（非纯文本数据）；
- 支持上传文件；
- 支持`formData`对象；
- 可以获取传输进度；
- 可以设置超时时间；

## 新增事件
- `xhr.onloadstart`: 绑定 HTTP 请求发出的监听函数；
- `xhr.onload`:  绑定请求成功完成的监听函数；
- `xhr.onerror`：绑定请求失败的监听函数；
- `xhr.onabort`:  绑定请求中止（调用了`abort()`方法）的监听函数；
- `xhr.onloadend`:  绑定请求完成（不管成功与失败）的监听函数；

示例：

```js
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

xhr.onloadstart = function () {
  console.log("onloadstart");
};

xhr.onload = function () {
  console.log("onload");
};

xhr.onerror = function () {
  console.log("onerror");
};

xhr.onabort = function () {
  console.log("onabort");
};

xhr.onloadend = function () {
  console.log("onloadend");
};

xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status < 304) {
      console.log("success", xhr.responseText);
    }
  }
};

xhr.open("POST", "/user/info", true);
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.send("a=1&b=2");
```

## 进度事件
进度`progress()`事件会在请求接收到数据的时候周期性触发。

该事件有以下几个属性：

- `lengthComputable`表示进度是否可以计算；
- `loaded`表示已完成的数据；
- `total`表示数据总量；

示例：

```js
var xhr = new XMLHttpRequest();

// 下载一张图片
xhr.open("GET", "https://via.placeholder.com/600x400", true);
// 以 Blob 方式接收二进制数据
xhr.responseType = "blob";

// 监听进度
xhr.onprogress = function (event) {
  if (event.lengthComputable) {
    let percent = ((event.loaded / event.total) * 100).toFixed(2);
    console.log(`下载进度: ${percent}% (${event.loaded} / ${event.total} 字节)`);
  } else {
    console.log(`已下载: ${event.loaded} 字节`);
  }
};

// 请求完成
xhr.onload = function () {
  if (xhr.status === 200) {
    console.log("下载完成");

    // 创建 URL 并展示图片
    let imgURL = URL.createObjectURL(xhr.response);
    let img = document.createElement("img");
    img.src = imgURL;
    document.body.appendChild(img);
  }
};

// 监听错误
xhr.onerror = function () {
  console.error("下载失败");
};

// 发送请求
xhr.send();
```

## 超时控制
`xhr`对象增加了一个`timeout`属性，用于表示发送请求后等待多少毫秒，如果响应不成功就中断请求，当请求超时后会触发`ontimeout`事件。

示例：

```js
var xhr;
if (window.XMLHttpRequest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}

xhr.ontimeout = function() {
  alert("Request did not return in a second.");
};

xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status < 304) {
      console.log("success", xhr.responseText);
    }
  }
};

xhr.open("POST", "/user/info", true);
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.timeout = 1000; // 设置 1 秒超时
xhr.send("a=1&b=2");
```

## FormData 类型
`FormData`类型便于表单序列化，也便于创建与表单类似格式的数据然后通过`xhr`发送。

该类型有一个`append()`方法，接收两个参数：键和值，相当于表单字段名称和该字段的值。

示例：

```js
let data = new FormData();
data.append("name", "Nicholas");
```

有了`FormData`实例，可以直接传给`xhr`对象的`send()`方法，使用`FormData`不再需要给`xhr`对象显式设置任何请求头部了。`xhr`对象能够识别作为`FormData`实例传入的数据类型并自动配置相应的头部。

示例：

```js
var xhr = new XMLHttpRequest();

xhr.onreadystatechange = function() {
  if (xhr.readyState == 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
      alert(xhr.responseText);
    } else {
      alert("Request was unsuccessful: " + xhr.status);
    }
  }
};

xhr.open("post", "postexample.php", true);
let form = document.getElementById("user-info");
xhr.send(new FormData(form));
```
