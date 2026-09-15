---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/01-bom/04-history/","dg-note-properties":{}}
---

`window.history`表示浏览器当前窗口访问的历史纪录

```js
window.history;
// window 也可以省略
history;
```

`history`的属性和方法：

| 属性/方法 | 说明 |
| --- | --- |
| length | 返回历史列表中的网址数 |
| back() | 加载 history 列表中的前一个 URL |
| forward() | 加载 history 列表中的下一个 URL |
| go() | 加载 history 列表中的某个具体页面 |

```js
console.log(history.length);
history.back();
history.forward();
history.go(1); // 正数的时候表示下一个 URL
history.go(-1); // 负数的时候表示前一个 URL
```
