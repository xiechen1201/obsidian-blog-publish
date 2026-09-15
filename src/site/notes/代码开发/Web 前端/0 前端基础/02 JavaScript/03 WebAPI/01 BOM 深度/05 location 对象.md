---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/01 BOM 深度/05 location 对象/","dg-note-properties":{}}
---

---
---
`window.location`表示当前页面的`URL`信息。

```js
window.location;
// window 也可以省略
location;
```

## 🔢 简单认识 URL
先简单对`URL`进行认识。

`URL（uniform resource locator）`：统一资源定位器。

<br/>

用百度的`URL`进行举例 🌰：

```plain
这是当我去搜索 javascript 时候页面的 URL
https://www.baidu.com/s?tn=39042058_40_oem_dg&ie=utf-8&wd=javascript

https:// 协议
www.baidu.com 域名
:443 端口号，不显示（http 默认是:80，https默认是:443）
/s 具体的文件夹
?tn=39042058_40_oem_dg&ie=utf-8&wd=javascript 请求参数
#hash 哈希值（这里没有用到）
```

## 🔢 属性
`window.location`通过相应的属性可以获得`URL`信息

| 属性 | 说明 |
| --- | --- |
| `href` | 获取整个`URL` |
| `protocol` | 获取`URL`中的协议 |
| `hostname` | 获取`URL`中的域名 |
| `port` | 获取`URL`中的端口号 |
| `pathname` | 获取`URL`中的文件路径 |
| `search` | 获取`URL`中的参数 |
| `hash` | 获取`URL`中的哈希值 |

以上属性都是可读可写的！！！

```js
window.location.href; // 获取
window.location.href = "https://www.taobao.com"; // 设置

window.location.protocol; // 获取
window.location.protocol = "https://"; // 设置
```

> 📌  `window.onhashchange`事件可以在`URL`中的`hash`更改后触发。
>
> 用更改`hash`更改后不会刷新页面的特性还可以实现单页面应用（`SPA：Single-page Application`），简单说就是实际上只有一个页面来模拟多个页面之间的切换。
>

## 🔢 方法
| 方法 | 说明 |
| --- | --- |
| `assign()` | 更改`URL`地址，一般用`window.location.href = "xxx"`代替 |
| `replace()` | 将`URL`替换，不会新增历史纪录，调用`replace()`后，用户不能回退到前一页<br/>参数1：要替换的`url` |
| `reload()` | 将`URL`重新加载<br/>参数：`true`，可选，表示强制加载，不会从缓冲中加载页面 |

```js
window.location.assign("https://www.baidu.com");
window.location.replace("https://www.baidu.com");
window.location.reload();
window.location.reload(true);
```
