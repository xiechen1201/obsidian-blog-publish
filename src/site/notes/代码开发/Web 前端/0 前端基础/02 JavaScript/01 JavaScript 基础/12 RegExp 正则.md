---
{"dg-publish":true,"permalink":"//web/0/02-java-script/01-java-script/12-reg-exp/","dg-note-properties":{}}
---

## 🔢 转义符号 & 转义字符
在`javaScript`字符串中`\`表示「转义」！！！

在`\`后面跟上一个特定的符号或者字母就表示「转义字符」！！！

举例 🌰：

在`javaScript`的字符串中`""`内是不能继续使用`""`的（或者`''`内不能继续使用`''`），那我就想把「大神」用`""`引起来，这样就可以用`\`将字符串内的`""`进行转义。

```js
var str = "我是\"大神\"程序员";
console.log(str);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1655359536739-9b1d9e83-f7f6-4844-847e-2ca3b6dbf8f3.png)

或者使用`//`将「大神」括起来

```js
var str = "我是\\大神\\程序员";
console.log(str);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1655359642801-070d4a8d-92f9-4351-9200-d3bb6b8c9868.png)

`\`还有固定的组合（规定好的转义字符）：

- `\n`表示换行
- `\r`表示回车换行
- `\t`表示一个`Tab`按键的空格

```js
// \n 表示换行
var str = "我是\n大神程序员";
console.log(str);
document.write(str); // 页面显示的时候不会进行换行，转义字符是给编辑系统使用的，HTML是文档不会识别

// \r 表示回车换行
var str = "我是\r大神程序员";
console.log(str); // 效果不明显

// \t 表示一个 tab（四个空格），也称制表符
var str = "我是\t大神程序员";
console.log(str);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1655359818087-e5ca08b0-5895-4acb-8abc-a8a902ef030b.png)

另外还需要说一点的是在`javaScript`中的字符串默认是不容许换行的。

我们可以用`\`来将空格转化为空格。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1655359954288-4549e884-f033-4920-9ade-e5dab6e1e61d.png)

## 🔢 RegExp 正则表达式

`RegExp（regular expression）`正则表达式就是“将字符串按照一定的规则进行匹配或者检索这个规则中执行的字符类型”。

<br/>

「正则表达式」是一个对象，创建正则的方式有两种：

1、对象字面量（或者直接声明对象）

```js
// 类似创建对象和数组
// var obj = {}
// var arr = []

var reg = /正则/修饰符;
```

2、实例化对象

```js
// 类似创建对象和数组
// var obj = new Object()
// var arr = new Array()

var reg = new RegExp("正则","修饰符");
```

## 🔢 正则的引用关系
```js
// new 的时候是完全新的正则对象（深拷贝）
var reg1 = /test/;
var reg2 = new RegExp(reg1);
reg1.a = 1;
console.log(reg2.a); // undefind

// 不 new 的时候是拿来正则对象的引用
var reg1 = /test/;
var reg2 = RegExp(reg1);
reg1.a = 1;
console.log(reg2.a); // 1
```

## 🔢 正则的字符/括号
## 🔢 修饰符
正则表达式有以下修饰符

| `i` | 不区分大小写 |
| --- | --- |
| `g` | 全局匹配 |
| `m` | 多行匹配 |

```js
// i 是让正则在检测字符串的时候不区分大小写

// 用正则查找字符串中的 test
var reg = new RegExp("test");
var str = "This is a test";
// test() 是 RegExp 对象的一个方法，表示检测字符串中是否有匹配的字符
console.log(reg.test(str)); // true

// 用正则查找字符串中的 test
var reg = /test/;
var str = "This is a Test";
console.log(reg.test(str)); // false
// 如果我们不加修饰符得到 false ，因为字符串中没有小写的 test
// 当我们加上修饰符 i 就能得到

var reg = /test/i;
var str = "This is a Test";
console.log(reg.test(str)); // true
```

```js
// g 是让正则在检测字符串进行全局查找（默认在匹配成功一个之后就不会继续匹配）

// 不加修饰符 g
var reg = /test/i;
var str = "This is a Test; Test is important.";
console.log(str.match(reg)); // ['Test']

// 加了修饰符 g
var reg = /test/ig;
var str = "This is a Test; Test is important.";
console.log(str.match(reg)); // ['Test', 'Test']
```

```js
// m 是让正则在匹配的时候多行匹配（默认只会匹配开头）
// ^ 表示以...开头

// 不加 m
var reg = /^Test/gi;
var str = "Test is a function; \nTest is important;";
console.log(str.match(reg)); // ['Test']

// 加了 m
var reg = /^Test/gim;
var str = "Test is a function; \nTest is important;";
console.log(str.match(reg)); // ['Test', 'Test']
```

## 🔢 元字符（转义字符）
| `\w` | 表示任意一个字母、数字或下划线 |
| --- | --- |
| `\W` | 表示非... |
| `\d` | 表示 0 - 9 任意一个数字 |
| `\D` | 表示非... |
| `\s` | 表示 `\r`或`\n`或`\t`或`\v`或`\f`的空白字符 |
| `\S` | 表示非... |
| `\b` | 表示单词边界 |
| `\B` | 表示非... |

```js
// \w 表示任意一个字母、数字或下划线[0-9A-z_]
// \W 表示非...

var reg = /\wab/g;
var str = "234abc_%&";
console.log(str.match(reg)); // ['4ab']

// ========== 分割 ==========

var reg = /\Wab/g;
var str = "234abc_%&";
console.log(str.match(reg)); // null
```

```js
// \d 表示 [0-9]的数字
// \D 表示非...

var reg = /\dabc/g;
var str = "234abc_%&";
console.log(str.match(reg)); // ["4abc"]

// ========== 分割 ==========

var reg = /\Dabc/g;
var str = "234eabc_%&";
console.log(str.match(reg)); // ["eabc"]
```

```js
// \s 表示 [\r\n\t\v\f] 字符
// \S 表示非...
// \r回车 \n换行 \tTab \v垂直换行 \f换页

var reg = /\sab/g;
var str = "23 abc-&%\nabv";
console.log(str.match(reg)); // [' ab', '\nab']
```

```js
// \b 表示单词边界
// \D 表示非...
// 单词边届的意思：https://zhidao.baidu.com/question/584326176.html

var reg = /\bThis/g;
var str = "This is a test";
console.log(str.match(reg)); // ['This']
```

## 🔢 量词
> 个人对量词的理解📗：
>
> 正则默认只会按着一位进行匹配，量词是设置前面的字符按照`n`位进行匹配
>

💡正则表达式匹配的时候有两个特点需要切记：

1、不会回头在进行匹配：当一个或一组字符匹配成功后不会再进行匹配

2、贪婪模式：正则在进行匹配的时候能匹配多就不匹配少

总结：字符串从左到右依次先匹配多，再匹配少，如果一旦匹配上就不回头匹配了

<br/>

| `n+` | 表示`n`字符至少出现 1 次，最多出现无数次（等价`{1,}`） |
| --- | --- |
| `n*` | 表示`n`字符至少出现 0 次，最多出现无数次（等价`{0,}`） |
| `n?` | 表示`n`字符至少出现 0 次，最多出现 1 次（等价`{0,1}`） |
| `n{x, y}` | 表示`n`字符至少出现`x`次，最多出现`y`次 |
| `n{x}` | 表示`n`字符只能出现`x`次 |

```js
// n+

var reg = /\w+/g; // 匹配任意字符出现 {1,} 位
var str = "abcdefg";
console.log(str.match(reg)); // ['abcdefg']
```

```js
// n*

var reg = /\w*/g; // 匹配任意字符出现 {0,} 位
var str = "abcdefg"; // g 和 " 之间也会被匹配
console.log(str.match(reg)); // ['abcdefg', '']

// ========== 分割 ==========

var reg = /\d*/g; // 匹配任意数字出现 {0,} 位
var str = "abcdefg"; // 当匹配不到数字的时候就会去匹配空格，例如 a 和 b 之间的空格，b 和 c 之间的 空格
console.log(str.match(reg)); // ['', '', '', '', '', '', '', '']
```

```js
// n?

var reg = /\w?/g; // 匹配任意字符，出现 {0,1} 位
var str = "abcdefg";
console.log(str.match(reg)); //  ['a', 'b', 'c', 'd', 'e', 'f', 'g', '']
```

```js
// n{x,y}

var reg = /\w{1,2}/g; // 匹配任意字符出现 {1,2} 位
var str = "abcdefg";
console.log(str.match(reg)); // ['ab', 'cd', 'ef', 'g']
```

```js
// n{x,}

var reg = /\w{5,}/g;
var str = "abcdefg";
console.log(str.match(reg)); // ['abcdefg']
```

```js
// n{x}

// 匹配 138 开头的11位手机号码
var str = "13812345678";
var reg = /^138\d{8}/;
console.log(reg.test(str));
```

## 🔢 单字符
| `|` | 表示或者，可以和`()`组合 |
| --- | --- |
| `.` | 表示任意一个非空白字符 |
| `^n` | 表示以`n`开头的字符串 |
| `n$` | 表示以`n`结尾的字符串 |

```js
// ｜ 表示或者

var str = "234asjkhjsais12312lajknkj";
var reg = /(123|234)/g;
console.log(str.match(reg)); // [234, 123]

// ==========

// 在（）内使用 ｜ 表示 123[a-z] 或者 234[a-z]
var str = "234vajkhjsais123slajknkj";
var reg = /(123|234)[a-z]/g;
console.log(str.match(reg)); // ['234v', '123s']

// ==========

// 直接使用 ｜ 表示 123 或者 234[a-z]
var str = "234vajkhjsais123slajknkj";
var reg = /123|234[a-z]/g;
console.log(str.match(reg)); // ['234v', '123']
```

```js
// . 表示任意一个非空白字符
var reg = /./g;
var str = "This\ris\ta\ntest;6&";
console.log(str.match(reg)); // ['T', 'h', 'i', 's', 'i', 's', '\t', 'a', 't', 'e', 's', 't', ';', '6', '&']
```

```js
// ^n

var reg = /^ab/gm;
var str = "abcdabcd\nabcdabcd";
console.log(str.match(reg)); // ['ab', 'ab']
```

```js
// n$

var reg = /cd$/gm;
var str = "abcdabcd\nabcdabcd";
console.log(str.match(reg)); // ['cd', 'cd']
```

```js
// 检查以abcd开头和以abcd结尾的字符串
var reg = /^abcd.*abcd$/g;
var str = "abcd123123abcd";
console.log(str.match(reg)); // ['abcd123123abcd']

// ========== 分割线 ==========

// 匹配检查以abcd开头和以abcd结尾的字符串，且中间是数字
var reg = /^abcd\d+abcd$/g;
var str = "abcd123123abcd";
console.log(str.match(reg)); // ['abcd123123abcd']
```

## 🔢 正向预查和反向预查
相关链接：

[javascript正则表达式---正向预查 - chenby - 博客园](https://www.cnblogs.com/dh-dh/p/5261044.html)

| `n(?=z)` | 正向匹配，表示`n`后紧挨着`z`的字符串，`z`不会出现在匹配结果字符串里面 |
| --- | --- |
| `n(?!z)` | 反向匹配，表示`n`后非紧挨着`z`的字符串，`z`不会出现在匹配结果字符串里面 |
| `n(?:z)` | 不捕获分组，不让字符串捕获子表达式 |

```js
// n(?=z)

var str = "abcdabcd";
var reg = /a(?=b)/g; // 匹配 a 后面是 b 的字符串，b 不会出现在匹配的字符串里
console.log(str.match(reg)); // ['a', 'a']

// ========== 分割 ==========

var str = "windows98 windows7 windows8 windows8.1 windows10 windowsVista windowsXP";
var reg = /windows(?=7|8\.1|8)/g;
var a = str.match(reg);
console.log(a); //返回[ 'windows', 'windows', 'windows' ]

// ========== 分割 ==========
// 对密码进行校验，密码的规则：至少 6 位，必须包含数字、大写字母、小写字母、特殊符号
// reg 有点多条件并列的意思 if(6位 && 包含数字 && ...)
var reg = /^.*(?=.{6,})(?=.*\d)(?=.*[A-Z])(?=.*[a-z])(?=.*[!@#$%^&])/;
var str = "123abcABC^";
reg.test(str);  // true
```

```js
// n(?!z)

var str = "windows98 windows7 windows8 windows8.1 windows10 windowsVista windowsXP";
var reg = /windows(?!7|8\.1|8)/g;
var a = str.match(reg);
console.log(a); //返回[ 'windows', 'windows', 'windows' ,'windows']
```

```js
// n(?:z)

var str = "abc";
var reg = /(a)(b)(c)/;
console.log(str.match(reg));  // ['abc', 'a', 'b', 'c']
// 用子表达式括起来，不仅能匹配(b)(c)还能单独匹配 (b) 和 (c)
// 这样的行为称为 捕获分组（捕获子表达式）

// ========== 分割 ==========

var str = "abc";
var reg = /(?:a)(b)(c)/;
console.log(str.match(reg));  // ['abc', 'b', 'c']
```

## 🔢 括号表达式
| `[]` | 表示只要是括号内的任意一位 |
| --- | --- |
| `()` | 表达式的引用，一般和表达式一起使用（或者理解为一个分组） |

```js
// [] 表示只要是括号内的任意一位

var str = "0917sqjnka12198";
var reg = /[1234567890][1234567890][1234567890]/g;
console.log(str.match(reg)); // ['091', '121']

// 以上的正则表示字符串中的第1位是[]里的任意1位，且第2位是[]里的任意1位，且第3位是[]里的任意1位
// 匹配成功的字符不会再参与匹配
// 比如：091 先进行匹配，如果 091 匹配成功，接着用 7sq 进行匹配，7sq 不匹配在用 sqj 进行匹配...

// ========== 分割 ==========

var str = "wxyz";
var reg = /[wx][xy][z]/g;
console.log(str.match(reg)); // [xyz]
// 第一次：wxy 去匹配，不成功因为第3位要求是z
// 第二次：xyz 去匹配，成功
```

```js
// [] 内默认是或的关系，内部还可以用 - 表示区间
var str = "ankjkjjkGhhjqwhq9Z087Jhjhjgu";
// var reg = /[a-z][0-9][A-Z]/g;
var reg = /[a-z0-9A-Z][0-9][A-Z]/;
console.log(str.match(reg)); // ["q9Z"]
```

```js
// [] 内使用 ^ 表示非的意思
var str = "ankjkjjkGhhjqwhq9Z087Jhjhjgu";
var reg = /[^0-9][a-z]/g;
console.log(str.match(reg)); // ['an', 'kj', 'kj', 'jk', 'Gh', 'hj', 'qw', 'hq', 'Jh', 'jh', 'jg']
```

```js
// 子表达式和反向引用
// 例如下面的正则：匹配 xxyy yyzz 这样的迭代字符

// 当我们想要匹配任意两个相同字符的时候，就可以使用 () 表面子表达式
var str = "bbaaaaccaaaaidddbaaaaa";
var reg = /(\w)\1(\w)\2/g;
// () 内就是一个表达式，\1 表示引用第一个表达式，也就是 (\w)
// \2 表示引用第二个表达式，也就是 (\w)

// (\w)(\w) 和 (\w)\1 的区别：
// (\w)(\w) 表示任意两个连续的字符，比如 Ac、MM、K9 都属于两个连续的 \w 字符
// \1 则表示对 (\w) 的引用，当 (\w) 匹配到 b 的时候，\1 也表示 b

console.log(str.match(reg)); // ['bbaa', 'aacc', 'aaaa', 'aaaa']
```

## 🔢 正则对象的属性
正则表达式是个对象，所以它有相关的属性。

```js
var reg = /(\w)\1(\w)\2/g;
console.log(reg.global); // 是否设置 g 修饰符
console.log(reg.ignoreCase); // 是否设置 i 修饰符
console.log(reg.multiline); // 是否设置 m 修饰符
console.log(reg.source); // 正则表达式的本体
```

## 🔢 正则对象的方法
## 🔢 reg.test(str)

用正则去检测字符串是否符合正则的规则！

<br/>

```js
// 匹配 138 开头的11位手机号码
var str = "13812345678";
var reg = /^138\d{8}/;
console.log(reg.test(str));
```

## 🔢 reg.exec(str)

根据正则表达式查找，结果会返回一个长度为1的数组 （数组只有一个值）

每次调用后递增结果，递增到最后返回`null`然后从头递增

<br/>

```js
var str = "This is Test. Test is important"
var reg = /Test/g
console.log(reg.exec(str));
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1655708244477-283eaf50-0d1d-4d13-9950-ee9d44f820a7.png)

<br/>color3
另外字符串还有`match()`、`replace()`可以和正则搭配使用。

详见：

<br/>

[String 字符串相关的方法](https://www.yuque.com/xiechen/px9euv/uulxg4)

## 🔢 贪婪模式和非贪婪模式
正则表达式默认是「贪婪模式」（能匹配多绝不匹配少）。

比如我们想匹配`{{}}`的字符串：

```js
var str = "abc{{efg}}abcd{{xyz}}";
var reg = /{{.*}}/g;
console.log(str.match(reg)); // ['{{efg}}abcd{{xyz}}']
```

可以看到默认会直接从`{{e`然后匹配到`z}}`，这就是贪婪模式的表现。

那么如何将「贪婪模式」转为「非贪婪模式」呢？

```js
var str = "abc{{efg}}abcd{{xyz}}";
var reg = /{{.*?}}/g;
console.log(str.match(reg)); //  ['{{efg}}', '{{xyz}}']
```

我们只需要在`{{.*}}`加上问号`{{.*?}}`就变为非贪婪模式了！！！

`?`单独使用的时候是表示字符至少出现 0 次，最多出现 1 次，而`?`和[量词](https://www.yuque.com/xiechen/px9euv/klhpk2/edit#RoWb1)一起使用的时候就会变成「非贪婪模式」

举例 🌰：

```js
var str = "aaaaaa";
var reg = /\w?/g;
console.log(str.match(reg)); // ['a', 'a', 'a', 'a', 'a', 'a', '']

// ========== 分割 ==========

var str = "aaaaaa";
var reg = /\w??/g;
console.log(str.match(reg)); // ['', '', '', '', '', '', '']
```

## 🔢 和 String.prototype.replace() 联合的案例：

该方法用来替换字符串中的字符，接收两个参数

参数1：要替换的字符串或正则表达式

参数2：要将字符串替换为什么字符，参数2还可以是一个函数，函数的参数是正则中的子表达式

💡 该方法本身不具备全局替换的能力，默认只会替换一次

<br/>

```js
// 将 plus 替换为 +
var str = "JSplusplus";
console.log(str.replace("plus", "+")) // JS+plus

// 用正则的方式去替换
var str = "JSplusplus";
console.log(str.replace(/plus/g, "+")) // JS++
```

```js
// 用 $ 去拿正则中的子表达式
var str = "aabbccdd";
var reg = /(\w)\1(\w)\2/g;
console.log(str.replace(reg, "$2$2$1$1")); // bbaaddcc

// 这里的 $2 表示正则中的 (\w)\2 中的 (\w)
// $1 表示正则中的 (\w)\1 中的 (\w)
```

```js
// 用 replace 的函数去实现

var str = "aabbccdd";
var reg = /(\w)\1(\w)\2/g;
var str1 = str.replace(reg, function ($, $1, $2) {
  console.log($, $1, $2); // aabb a b
                          // ccdd c d
  // 每次匹配成功都会执行一次 replace 方法
  // $ 当前匹配出来的字符串 aabb 和 ccdd
  // $1 是第一个子表达式
  // $2 是第二个子表达式
  return $2 + $2 + $1 + $1;
});
console.log(str1); // bbaaddcc
```

```js
// 将 -p 转为大写
var str = "js-plus-plus";
var reg = /\-p/g;
var reg = str.replace(reg, "P");
console.log(reg); // jsPlusPlus

// 将 -b 和 -p 都转化为大写
var str = "js-blus-plus";
var reg = /\-(\w)/g;
var reg = str.replace(reg, function ($, $1) {
  // $1 就表示 (\w) 也就是 b 和 p
  return $1.toUpperCase();
});
console.log(reg);

// Plus 转换为 _plus
var str = "jsPlusPlus";
var reg = /([A-Z])/g;
var res = str.replace(reg, function ($, $1) {
  return "_" + $1.toLowerCase();
});
console.log(res); // js_plus_plus
```

```js
// 将 xxyyzz 转为 XxYyZz

var str = "xxyyzz",
    reg = /(\w)\1(\w)\2(\w)\3/,
    res = str.replace(reg, function ($, $1, $2, $3) {
      return $1.toUpperCase() + $1 + $2.toUpperCase() + $2 + $3.toUpperCase() + $3;
    });
console.log(res); // XxYyZz
```

```js
// 将 aabbcc 转为 a$b$c$ 且不能使用 replace 的 function
var str = "aabbcc",
    reg = /(\w)\1(\w)\2(\w)\3/,
    res = str.replace(reg,`$1$$$2$$$3`); // 当使用 $ 的时候需要用 $ 进行转义，其他字符正常
console.log(res); // a$b$c

// ========== 分割 ==========

var str = "aabbcc",
    reg = /(\w)\1(\w)\2(\w)\3/,
    res = str.replace(reg,`$1/$2/$3`);
console.log(res); // a/b/c
```

```js
// 正则给字符串去重

var str = "aabbcc",
    reg = /(\w)\1(\w)\2(\w)\3/g;
console.log(str.replace(reg, "$1$2$3")); // abc

// ========== 分割 ==========

var str = "aaaaabbbbbcccccccc",
reg = /(\w)\1*/g;
console.log(str.replace(reg, "$1")); // abc
```

```js
// 给字符用千分位用,分割
// 100000000000 =》 100,000,000

var str = "1000000000000";
// (\B)(\d{3}) 都属于 ?= 的条件！
var reg = /(?=(\B)(\d{3})+$)/g;
console.log(str.replace(reg, "$1,")); // 1,000,000,000,000
```

```js
// 用 replace 给模版字符串替换数据

var str = "My name is {{name}}. I'm {{age}} years old.";
var reg = /{{(.*?)}}/g;
var res = str.replace(reg, function ($, $1) {
  return {
    name: "Jone",
    age: 32,
  }[$1];
});

console.log(res); // My name is Jone. I'm 32 years old.
```
