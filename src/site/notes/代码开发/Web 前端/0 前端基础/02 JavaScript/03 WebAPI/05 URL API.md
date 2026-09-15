---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/05-url-api/","dg-note-properties":{}}
---

在没有 URL API 的时候，对于 URL 的解析操作都需要手动的拼接，或者通过正则进行匹配等等。

有了这个 API 后操作 URL 变得非常轻松。

## 🔢 URL 对象
使用`URL()`创建一个 URL 对象，它接收两个参数。

- url，字符串形式的 URL；
- base（可选），字符串或者其他 URL 对象，作为解析相对 URL 的基准；

```js
const baseURL = new URL('https://example.com/base/path/');
const relativeURL = new URL('relative/path', baseURL);

console.log(relativeURL.toString());
// https://example.com/base/path/relative/path
```

URL 对象提供了很多属性，用于读取或修改 URL 中的内容：

- `href`字符串形式的完整 URL；
- `protocol`协议，包括末尾的`:`，例如`'http:'`
- `username` URL 中的用户名；
- `password` URL 中的密码；
- `host`主机名 + 端口号组合，例如`'example.com:8080'`；
- `hostname`域名或者 IP 地址；
- `port`端口；
- `pathname` URL 中的路径；
- `search` URL 中的查询字符串，包括前置的`?`；
- `hash`URL 中的片段字符串，包括前置的`#`；
- `searchParams`只读的`URLSearchParams`对象，可用于操作查询字符串；

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776771203573-0d54baac-c0c0-4d72-ae96-4677e1f14c38.png)

## 🔢 URLSearchParams 对象
`URLSearchParams`对象提供了一组标准的 API 方法，通过它们可以检查和修改查询字符串。

两种方式得到`URLSearchParams`对象：

1. 通过 URL 对象的`searchParams`属性；
2. 通过实例化`new URLSearchParams()`并传入查询字符串；

`URLSearchParams`具有以下方法：

- `append(name, value)`用指定的`name`和`value`追加新的查询参数；
- `delete(name)`删除指定`name`的查询参数；
- `get(name)`查询指定`name`的查询参数，如果没有返回`null`；
- `getAll(name)`查询指定`name`的匹配的所有值，如果没有则返回空数组；
- `has(name)`查询指定`name`的查询参数是否存在，返回布尔值；
- `set(name, value)`用指定的`name`和`value`来设置/更新查询参数，如果没有则为新增；
- `sort()`对参数字符串按照 Key 进行排序（只会按照 Key 进行排序，例如`b=2&a=10&a=1`===>`a=10&a=1&b=2`）；

示例：

```js
let qs = "?q=javascript&num=10";
let searchParams = new URLSearchParams(qs);

alert(searchParams.toString()); // " q=javascript&num=10"

searchParams.has("num"); // true
searchParams.get("num"); // 10
searchParams.set("page", "3");
alert(searchParams.toString()); // " q=javascript&num=10&page=3"

searchParams.delete("q");
alert(searchParams.toString()); // " num=10&page=3"
```

`URLSearchParams`的实例是可迭代对象：

```js
let qs = "?q=javascript&num=10";

let searchParams = new URLSearchParams(qs);

for (let param of searchParams) {
  console.log(param);
}

// ["q", "javascript"]
// ["num", "10"]
```
