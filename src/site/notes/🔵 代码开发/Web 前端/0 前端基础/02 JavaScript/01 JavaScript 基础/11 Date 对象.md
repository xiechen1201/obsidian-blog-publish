---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/11 Date 对象/","dg-note-properties":{}}
---

---
---
## 🔢 创建 Date
`Date`对象表示时间，用实例化`Date`函数来创建时间。

`new Date()`返回一个时间片段，不会实时更新！！！

```js
var now = new Date();
```

## 🔢 Date 的参数
在不给`new Date()`传递参数的情况下，`new Date()`返回当前系统的时间。

```js
var now = new Date();
console.log(now); // Mon Jul 11 2022 09:50:22 GMT+0800 (中国标准时间)
```

另外`Date()`还支持以下参数，来生成指定的时间片段：

```js
new Date(时间戳);
new Date(2022, 07, 11, 09, 00, 00); // 返回 2022 年 7 月 11 日 9 时 0 分 0 秒的时间
new Date("2022/07/11 09:00:00");
new Date("2022-07-11 09:00:00");
```

## 🔢 Date 的方法
## 🔢 toLocaleString()、toString()、valueOf() 和 toUTCString()
`Date` 类型重写了 `toLocaleString()`、`toString()`和 `valueOf()` 方法。

- `new Date().toLocaleString()` 方法返回与浏览器运行的本地环境一致的日期和时间（例如 2/1/2019 12:00:00 AM）。
- `new Date().toString()` 方法通常返回带时区信息的日期和时间，而时间也是以 24 小时制（0~23）表示的（例如`Thu Feb 1 2019 00:00:00 GMT-0800 (Pacific Standard Time)`）。

另外由于各个浏览器存在显示的差异所以 `toLocaleString()` 和 `toString()` 可能只对调试有用， 不能用于显示。

- `new Date().valueOf()` 方法被重写后返回的是日期的毫秒（例如 1622704543951）。
- `new Date().toUTCString()`方法返回的是 UTC 时间。

## 🔢 getTime() / setTime()

`getTime()`用于返回一个时间戳，和`valueOf()`结果一致

`setTime()`用于设置一个时间戳

<br/>

> 什么是时间戳？
>
> 时间戳指的是一个纪元时间，也就是 1970 年 1 月 1 日 0 点 0 分 0 秒到一个`date`之间相差的毫秒数，利用时间戳可以对比两个时间的大小等操作。
>
> 返回的时间戳是 UTC 的时间戳，全世界都一样，不过在使用`new Date(timestamp)`实例化的时候，该方法会自动转换为本地时间。
>

```js
new Date().getTime(); // 获取当前系统时间的时间戳（也就是当前时间到 1970 年 1 月 1 日 0 点 0 分 0 秒过了多少毫秒）
new Date("2022/07/12 23:59:59").getTime(); // 获取 7 月 12 日的时间戳

new Date().setTime(1657505268625);
```

## 🔢 获取/设置年月日时分秒毫秒
| 方法 | 说明 |
| --- | --- |
| `getFullYear()`/`setFullYear()` | 获取/设置日期的年 |
| `getMonth()`/`setMonth()` | 获取/设置日期的月（返回 0- 11） |
| `getDate()`/`setDate()` | 获取/设置日期的日（返回 1-31） |
| `getDay()`/`setDay()` | 获取/设置日期的星期（返回 0-6，0 表示星期日） |
| `getHours()`/`setHours` | 获取/设置日期的时 |
| `getMinutes()`/`setMinutes()` | 获取/设置日期的分 |
| `getSeconds()`/`setSeconds()` | 获取/设置日期的秒 |
| `getMilliseconds()`/`setMilliseconds()` | 获取/设置日期的毫秒 |

## 🔢 实例
## 🔢 自定义时间格式
```js
var d = new Date("1998-09-09");   // 没有时分秒,默认是早上8点
```

## 🔢 设置时间
```js
var now = new Date();
// 设置9天后的时间
now.setDate( now.getDate() + 9 );
```

## 🔢 时间差
```js
function diff(start,end){
  //返回时间差 单位 秒
  return Math.abs(start.getTime() - end.getTime()) / 1000;
}
```

## 🔢 倒计时
```js
var showtime = function () {
  var nowtime = new Date(),  //获取当前时间
      endtime = new Date("2020/8/8");  //定义结束时间
  var lefttime = endtime.getTime() - nowtime.getTime(),  //距离结束时间的毫秒数
      leftd = Math.floor(lefttime/(1000*60*60*24)),  //计算天数
      lefth = Math.floor(lefttime/(1000*60*60)%24),  //计算小时数
      leftm = Math.floor(lefttime/(1000*60)%60),  //计算分钟数
      lefts = Math.floor(lefttime/1000%60);  //计算秒数
  return leftd + "天" + lefth + ":" + leftm + ":" + lefts;  //返回倒计时的字符串
}
```
