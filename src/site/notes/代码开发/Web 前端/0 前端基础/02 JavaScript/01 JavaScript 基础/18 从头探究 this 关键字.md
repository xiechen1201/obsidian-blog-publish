---
{"dg-publish":true,"permalink":"//web/0/02-java-script/01-java-script/18-this/","dg-note-properties":{}}
---

`this`对象表示当前环境的上下文，在以下场景中`this`指向有所不同。

1、普通函数内的`this`

普通函数内部的`this`对象指向`window`

```js
function test() {
  this.a = 1;
  console.log(this); // window
}
test();
```

```js
var a = 1;
function test() {
  console.log(this); // window
  console.log(this.a); // 1，相当于 window.a
}
test();
```

```js
function test(a) {
  this.a = a;
  console.log(this); // window
  return this;
}
console.log(test(1).a); // 1，相当于 window.a
```

严格模式下，函数内的`this`是`undefind`

```js
var a = 1;
function test() {
  "use strict";
  console.log(this); // undefind
  console.log(this.a); // error
}
test();
```

2、对象方法内的`this`指向对象本身

```js
var obj = {
  a: 1,
  test: function () {
    console.log(this); // obj
  },
};
obj.test();
```

3、实例化构造函数的时候`this`指向实例化对象

```js
function Test(a) {
  this.a = a;
  console.log(this); // 实例化对象 {a: 1}
}

new Test(1)
```

实例化对象调用构造函数原型方法的时候`this`仍然指向实例化对象

```js
function Test(a) {
  this.a = a;
}
Test.prototype.say = function () {
  console.log(this.a); // 1，this 指向实例化对象
};

var test = new Test(1);
test.say();
```

4、事件处理程序中的`this`指向`DOM`本身

```js
var oBtn = document.getElementById("btn");
oBtn.onclick = function () {
  console.log(this); // this 指向 button
};
```

5、定时器中的`this`指向`window`

```js
var oBtn = document.getElementById("btn");
oBtn.onclick = function () {
  console.log(this); // this 指向 button
  var _this = this;

  setTimeout(function () {
    console.log(this); // setTimeout 内的 this 指向 window
    console.log(_this); // this 指向 button
  }, 1000);
};
```

6、模块化中的`this`

```js
var initModule = (function () {
  return {
    a: 1,
    b: 2,
    plus: function () {
      return this.a + this.b;
    },
  };
})();

console.log(initModule); // {a: 1, b: 2, plus: fn}
console.log(initModule.plus()); // 3
```

7、闭包中的`this`也指向`window`

```js
function test() {
  console.log(this); // window
  return function test1() {
    console.log(this); // window
  };
}
var test3 = test()
test3()
```

8、其余情况下，谁调用方法，则`this`就指向谁。

```js
function Test(a) {
  this.a = a;
}
Test.prototype = {
  a: 2,
  say: function () {
    console.log(this.a);
  }
}

var test = new Test(1);
test.say(); // 1，实例化调用的时候，this 指向实例化对象
Test.prototype.say(); // 2，函数原型直接调用的时候 this 指向 prototype（也可以理解为对象方法内的 this 指向对象本身）
```

综合案例：

```js
(function () {
  function Test(a, b) {
    this.oBtn = document.getElementById("btn");
    this.a = a;
    this.b = b;
  }
  Test.prototype = {
    init: function () {
      // this 指向实例化对象
      this.bindEvent();
    },
    bindEvent: function () {
      // 事件处理程序中的 this 指向 DOM，所以需要改变 this 指向
      this.oBtn.onclick = this.plus.bind(this);
    },
    plus: function () {
      // this 指向实例化对象
      console.log(this.a + this.b);
    },
  };

  window.Test = Test;
})();

var test = new Test(1, 2).init();
```
