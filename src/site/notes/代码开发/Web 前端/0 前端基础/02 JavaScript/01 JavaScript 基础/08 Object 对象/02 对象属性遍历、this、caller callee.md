---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/08 Object 对象/02 对象属性遍历、this、caller callee/","dg-note-properties":{}}
---

---
---
## 🔢 链式操作
如何实现「对象.方法.方法.方法」这样的链式操作呢？

可以在方法内返回对象的形式。

```js
var sched = {
  wakeup: function () {
    console.log("跑步");
    return this;
  },
  morning: function () {
    console.log("去购物");
    return this;
  },
  noon: function () {
    console.log("休息一下");
    return this;
  },
};
sched.wakeup().morning().noon();
```

## 🔢 对象相关的方法和属性
## 🔢 for...in...

`for...in...`方法可以用来遍历对象。

<br/>

```js
var car = {
  brand: "Benz",
  color: "red",
  displacement: "3.0",
  lang: 5,
  width: 2.5,
};
for (const key in car) {
  console.log(car[key]); // "Benz" "red" "3.0" 5 2.5
}
```

`for...in...`遍历会把对象原型上的属性都打印出来！！！

```js
Object.prototype.name = "TestPrototypeAttribute"
var car = {
  brand: "Benz",
  color: "red",
  displacement: "3.0",
  lang: 5,
  width: 2.5,
};
for (const key in car) {
  console.log(car[key]); // "Benz" "red" "3.0" 5 2.5 TestPrototypeAttribute
}
```

<br/>warning
⚠️ 注意

`ECMAScript`中对象的属性是无序的，所有可枚举的属性都会返回一次，但返回的顺序可能会因浏览器而异。

<br/>

如果`for...in...`循环要迭代的变量是`null`或`undefined`，则不执行循环体：

```js
for (let item in null) {
  console.log(item); // 无输出
}

for (let item in undefined) {
  console.log(item); // 无输出
}
```

`for...in...`也可以用来遍历数组：

```js
var arr = [1, 2, 3, 4, 5];
for (const key in arr) {
  console.log(key); // 0 1 2 3 4
}
```

## 🔢 in 语句

`in`用来判断实例对象是否包含某个属性，该语句和`hasOwnProperty`最大的区别就是`in`语句会到实例对象的原型上查找属性（会到原型上寻找）。

<br/>

```js
var car = {
  brand: "Benz",
  color: "red",
};

// 类似 car["displacement"]
console.log("displacement" in car); // false
```

```js
Car.prototype.displacement = "3.0";
function Car() {
  this.brand = "Benz";
  this.color = "red";
}

var car = new Car();
console.log("displacement" in car); // true
```

## 🔢 instanceof 语句

`instanceof`语句用来判断某个对象是否是某个构造函数的实例对象（类似`A`对象的原型里到底有没有`B`构造函数的原型，原型链的关系）。

<br/>

`instanceof`的原理：

只要右边变量的`prototype`在左边变量的原型链上即可。因此，`instanceof`在查找的过程中会遍历左边变量的原型链，直到找到右边变量的`prototype`，如果查找失败，则会返回`false`。

[instanceof的原理你了解吗？_instanceof原理_xianghong_yang的博客-CSDN博客](https://blog.csdn.net/weixin_44761091/article/details/123410965)

手写`instanceof`：

```js
function myInstanceof(instance, target) {
  while (true) {
    if (instance.__proto__ === null) {
      return false;
    }
    if (instance.__proto__ === target.prototype) {
      return true;
    }
    instance = instance.__proto__;
  }
}

function Person() {}
let person = new Person();

console.log(myInstanceof(person, Person));
console.log(myInstanceof(person, Object));
```

利用`instanceof`判断实例对象是不是某个构造函数的实例：

```js
function Car() {}
var car = new Car();

console.log(car instanceof Car); // true
console.log(car instanceof Object); // true
console.log([] instanceof Array); // true
console.log([] instanceof Object); // true
console.log({} instanceof Object); // true
```

所以这就导致了关系不是特别的明确，因为任何对象的原型链顶端都是`Object.prototype`。

更靠谱的方式是调用`Object.prototype.toString()`方法。

```js
var a = [];
console.log(a.constructor);
console.log(a instanceof Array);
console.log(Object.prototype.toString.call(a)); // 使用 call 改变 this 指向为 a

if(Object.prototype.toString.call(a) === "[object Array]"){
  alert("是数组！")
}
```

因为`Array`构造函数重写了`toString`方法，所以我们只能调用`Object.prototype.toString()`方法。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1651820594341-bf12039b-e48c-413c-a2ca-b9cbf69a0d9f.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1651820548899-23640703-0a7b-4fb1-844e-ed6dc86ac296.png)

## 🔢 this 指向
再谈谈`this`指向的问题，只要记住：谁调用函数`this`就指向谁。

普通函数内部的`this`默认指向`window`：

```js
function test(b) {
  var a = 1;
  var b = b;
  function c() {}
  this.d = 3; // window.d = 3
}

test(123);
console.log(d); // 3，函数外部是可以直接访问到 d 变量的

// 预编译的过程
/**
 * AO = {
 *    this: window,
 *    argument: [123]
 *    a: undefind,
 *    b: undefind, => 123
 *    c: function
 * }
 * GO = {
 *    d: 3
 * }
 */
```

构造函数内部的`this`指向实例化对象：

```js
function Test() {
  this.name = 123;
}
var test = new Test();

/**
 * 实例化对象的执行过程
 *
 * 1、创建 this 对象
 * var this = {}
 *
 * 2、赋值原型
 * var this = {__proro__: Test.prototype}
 *
 * 3、属性赋值
 * var this = {__proro__: Test.prototype, name: 123}
 *
 * 4、返回 this 并指向实例化对象
 *
 */

// 预编译的过程
/**
 * Test 函数的 AO = {
 *    this: {window 对象} => {__proro__: Test.prototype, name: 123}
 * }
 * GO = {
 *    Test: function
 *    test: undefind => {name: 123}
 * }
 */
```

## 🔢 callee

`callee`的作用是指向当前所在函数的引用。

<br/>

```js
function test(a, b, c) {
  console.log(arguments.callee); // 指向当前所在的函数
  // ƒ test(a, b, c) { ... }

  console.log(arguments.callee.length); // 3，形参的长度
  console.log(test.length); // 3，形参的长度
  console.log(arguments.length); // 2，实参的长度
}
test(1, 2);
```

```js
function test1() {
  // 指向 test1 函数的引用
  console.log(arguments.callee); // ƒ test1() {
  test2()
}

function test2() {
  // 指向 test2 函数的引用
  console.log(arguments.callee); // ƒ test2()
}

test1()
```

某些时候在使用立即执行函数+递归时比较有用，比如实现函数实现 n-1 的累加：

```js
// 普通函数的实现方式
function test1() {
  console.log(arguments.callee);
  function test2() {
    console.log(arguments.callee);
  }
  test2()
}
test1()

// 立即执行函数的实现方式
var sum = (function (n) {
  if (n <= 1) {
    return 1;
  }
  // 当不知道函数名的时候如何递归？
  // 可以调用 callee 属性来拿到当前函数的引用
  return n + arguments.callee(n - 1);
})(10);
console.log(sum)
```

## 🔢 caller

`caller`属性返回函数被调用是所在的函数的引用。

<br/>

```js
test1();
function test1() {
  test2();
}

function test2() {
  // 返回当前被调用 test2 函数的函数引用
  // 必须放到一个执行函数内，放到全局无效
  console.log(test2.caller); // [Function: test1]
}

// 放在全局下无效
test3()
function test3() {
  console.log(test3.caller); // null
}
```
