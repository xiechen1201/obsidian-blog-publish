---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/10 String 常用方法/","dg-note-properties":{}}
---

---
---
[String - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)

## 🔢 charAt()

描述：从一个字符串中返回指定的字符

参数：下标

<br/>

```js
var anyString = "Brave new world";

console.log(anyString.charAt(0)); // B
console.log(anyString.charAt(1)); // r
```

## 🔢 indexOf()

描述：查找字符在字符串中的首次出现的下标

返回值：返回下标，如果字符串不存在则返回 -1

参数1：要被查询的字符串

参数2：开始查找的位置，默认是 0

<br/>

```js
"string".indexOf("i"); // 3
"string".indexOf("i", 2); // 3
```

## 🔢 lastIndexOf()

描述：查找字符在字符串中的最后出现的下标，

返回值：返回下标，如果字符串不存在则返回 -1

参数1：要被查询的字符串

参数2：开始查找的位置，默认是 0

<br/>

```js
'canal'.lastIndexOf('a'); // 3
'canal'.lastIndexOf('a', 2); // 1
```

## 🔢 concat()

描述：将一个或多个字符串进行合并

返回值：合并后的新字符串。

⭐️ 不会更改原字符串

参数：需要合并的字符串

<br/>

```js
let hello = 'Hello, '
console.log(hello.concat('Kevin', '. Have a nice day.'))
// Hello, Kevin. Have a nice day.
```

## 🔢 match()

描述：返回一个字符串匹配正则表达式的结果

返回值：返回与完整正则表达式匹配的所有结果

参数：正则

<br/>

```js
const paragraph = 'The quick brown fox jumps over the lazy dog. It barked.';
const regex = /[A-Z]/g;
const found = paragraph.match(regex);

console.log(found); // ["T", "I"]
```

## 🔢 search()

描述：返回正则在字符串中的下标

返回值：首次匹配项的索引，如果没有则返回 -1

<br/>

```js
var str = "hey JudE";
var re = /[A-Z]/g;
var re2 = /[.]/g;
console.log(str.search(re)); // 4
console.log(str.search(re2)); // -1
```

## 🔢 replace

描述：替换字符串中的字符

返回值：返回替换后的新字符串

⭐️ 不会更改原字符串

参数1：要替换的字符 ｜ 正则

参数2：新字符 ｜ `function`

<br/>

```js
const p = 'The quick brown fox jumps over the lazy dog.';
console.log(p.replace('dog', 'monkey'));
// The quick brown fox jumps over the lazy monkey.

const regex = /Dog/i;
console.log(p.replace(regex, 'ferret'));
// The quick brown fox jumps over the lazy ferret.
```

当指定参数2是一个函数的时，当匹配执行后，该函数就会执行。 函数的返回值作为替换字符串。

具体细节（和`String.prototype.replace()`联合的案例）：

[RegExp 正则](https://www.yuque.com/xiechen/px9euv/klhpk2#cjTVc)

## 🔢 split()

描述：按照指定的字符规则将字符串「分裂」为数组

参数：分隔符规则

返回：分隔后的数组

<br/>

```js
const str = 'The quick brown fox jumps over the lazy dog.';
console.log(str.split(' '));
// ['The', 'quick', 'brown', 'fox', 'jumps', 'over', 'the', 'lazy', 'dog.']
```

这和`Array.prototype.join()`是相反的：

```js
let arr = ['The', 'quick', 'brown', 'fox', 'jumps', 'over', 'the', 'lazy', 'dog.'];
console.log(arr.join(" "));
// The quick brown fox jumps over the lazy dog.
```

## 🔢 slice()

描述：提取某个字符串的一部分，

返回值：返回一个提取到的新字符串

⭐️ 不会改动原字符串

参数1: 开始提取的下标，容许为负数

参数2: 结束提取的下标（返回的字符不包括结束下标处的字符），如果没有参数2则一直提取到末尾，容许为负数

<br/>

```js
var str1 = 'The morning is upon us.'

console.log(str1.slice(1, 8)); // he morn
console.log(str1.slice(4, -2)); // morning is upon u
console.log(str1.slice(12)); // is upon us.
console.log(str1.slice(30)); // ""
```

```js
var str = 'The morning is upon us.';
str.slice(-3);     // 'us.'
str.slice(-3, -1); // 'us'
str.slice(0, -1);  // 'The morning is upon us'
```

## 🔢 substring()

描述：返回一个字符串在开始索引到结束索引之间的一个字符串

返回值：之间的新字符串

参数1: 开始截取的索引

参数2: 结束截取的索引（不包括）

- 如果参数1等于参数2，返回一个空字符串。
- 如果省略参数2 ，提取字符一直到字符串末尾。
- 如果任一参数小于 0 或为`NaN`，则被当作 0。
- 如果任一参数大于字符串的长度，则被当作字符串的长度
- 如果参数1大于参数2，执行效果就像两个参数调换了一样

<br/>

```js
var anyString = "Mozilla";

// 输出 "Moz"
console.log(anyString.substring(0,3));
console.log(anyString.substring(3,0));
console.log(anyString.substring(3,-3));
console.log(anyString.substring(3,NaN));
console.log(anyString.substring(-2,3));
console.log(anyString.substring(NaN,3));

// 输出 "lla"
console.log(anyString.substring(4,7));
console.log(anyString.substring(7,4));

// 输出 ""
console.log(anyString.substring(4,4));

// 输出 "Mozill"
console.log(anyString.substring(0,6));

// 输出 "Mozilla"
console.log(anyString.substring(0,7));
console.log(anyString.substring(0,10));
```

## 🔢 toLowerCase() / toUpperCase()

描述：将字符串转为小/大写形式

返回值：转换后的新字符串

<br/>

```js
console.log('中文简体 zh-CN || zh-Hans'.toLowerCase()); // 中文简体 zh-cn || zh-hans
console.log( "ALPHABET".toLowerCase()); // "alphabet"
```

```js
console.log('alphabet'.toUpperCase()); // 'ALPHABET'
```

## 🔢 trim() / trimStart() / trimEnd()

描述：删除字符串的两端/之前/之后的空白字符

返回：删除空白字符之后的新字符串

<br/>

```js
var orig = '   foo  ';
console.log(orig.trim()); // 'foo'
```

```js
var orig = '   foo  ';
console.log(orig.trimStart()); // 'foo  '
console.log(orig.trimEnd()); // '   foo'
```
