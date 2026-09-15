---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/19 call、apply 和 bind/","dg-note-properties":{}}
---

## call() 和 apply()
`call()`和`apply()`都是用于更改函数中`this`的指向。

```js
// 更改 this 的指向
function Car(brand, color) {
  this.brand = brand;
  this.color = color;
}
var newCar = {};

Car.call(newCar, "LiXiang", "black");
//Car.apply(newCar, ["LiXiang", "black"]);

console.log(newCar)
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/_assets/1650958414342-4f48f31c-02a1-43b1-8b9c-811726d89359.png)

`call()`和`apply()`的作用是一样的，他们的第一个参数都是要指向的对象，`call()`的第二个参数乃至第`N`个参数是传入的参数，`apply`的第二个参数是一个数组，也是要传入的参数。

<br/>

举例 🌰 ：

```js
function Compute() {
  this.plus = function (a, b) {
    console.log(a + b);
  };
  this.minus = function (a, b) {
    console.log(a - b);
  };
}
function FullCompute() {
  // 通过 call() 可以拿到 Compute 下面的函数
  Compute.call(this);
  this.mul = function (a, b) {
    console.log(a * b);
  };
  this.div = function (a, b) {
    console.log(a / b);
  };
}

var compute = new FullCompute();
compute.plus(1, 2);
compute.minus(1, 2);
compute.mul(1, 2);
compute.div(1, 2);
```

## 重写 call() 和 apply() 方法
重写的思路其实是基于对象调用方法，方法里的`this`指向该对象。

重写`call()`方法：

```js
// 写到函数的原型上，这样所有函数都能调用
Function.prototype.myCall = function (context) {
  // context 参数是要指向的 this 对象

  // 这样的判断不严谨
  if (typeof context === "object" && context !== null) {
    // 产生一个随机的属性名
    var prop =
        Math.random().toString(36).substr(3, 6) +
        new Date().getTime().toString(36);

    // 将参数 push 进一个新的数组里面
    var args = [];
    // i 从 0 开始是因为 0 是要指向的 this 对象而不是传递的参数
    for (var i = 1; i < arguments.length; i++) {
      args.push(arguments[i]);
    }

    // 给这个要指向的 this 对象新增一个属性，属性名就是生成的随机字符串
    // 属性值就是当前调用这个方法的 this ，也就是 test 函数
    context[prop] = this;

    // 因为 call 的参数是这样的 test.call({}, 1, 2)
    // 所以我们可以使用 eval() 将数组的参数进行结构 String([1, 2]) => 1, 2
    var res = eval("context[prop](" + args + ")");
    delete context[prop];

    // 将结果返回出去
    return res;
  }
};

// 测试一下
function test(a, b) {
  console.log(a, b); // 1 2
  console.log(this); // { name: "name1" }
  console.log(a + b); // 3
  return a + b;
}

test.myCall({ name: "name1" }, 1, 2);
```

重写`apply()`方法：

```js
Function.prototype.myApply = function (context) {
  if (typeof context === "object" && context !== null) {
    var prop =
        Math.random().toString(36).substr(3, 6) +
        new Date().getTime().toString(36);

    var args = [];
    // call 和 appley 只是接收的参数形式不同
    // 所以 arguments[1] 参数我们要取的参数
    for (var i = 0; i < arguments[1].length; i++) {
      args.push(arguments[1][i]);
    }

    context[prop] = this;
    var res = eval("context[prop](" + args + ")");
    delete context[prop];
    return res;
  }
};

// 测试一下
function test(a, b) {
  console.log(a, b); // 1 2
  console.log(this); // { name: "name2" }
  console.log(a + b); // 3
  return a + b;
}
test.myApply({ name: "name2" }, [1, 2]);
```

## bind()
`bind`也用于改变函数内的`this`指向。

```js
var p1 = {
  name: "张三",
  hobby: this.hobby,
  play: function (sex, age) {
    console.log("年龄" + age + "；性别" + sex + "；喜欢" + this.hobby);
  },
};
var p2 = {
  name: "李四",
  hobby: "踢球",
};

p1.play.call(p2, "男", 20); // 年龄20；性别男；喜欢踢球
p1.play.apply(p2, ["男", 20]); // 年龄20；性别男；喜欢踢球
p1.play.bind(p2, "男", 20)(); // 年龄20；性别男；喜欢踢球
```

`bind`和`call/apply`的区别：

`call/apply`改变函数的`this`后立即执行。

`bind`改变函数的`this`后不会立即执行，而是返回一个函数。

所以我们可以根据自己的需求来决定到底是用`call`、`apply`还是`bind`。

```js
var el = document.getElementById("box");
// 因为事件处理程序中的 this 指向元素本身
el.addEventListener("click", tabClick.bind(this), false);

function tabClick(){
  // do...
}
```

## bind 后的实例化
因为`bind`改变`this`后不会立即执行，所以它不会影响我们实例化构造函数。

```js
var p = {
  age: 18,
};
function Person() {
  console.log(this);
  console.log(this.age);
}

// bind(p) 改变的是 Person 函数内部的 this
// 也就是把 window 变成了 p
// 但是 new person2 的时候会产生新的 this 对象，所以没有影响
var person2 = Person.bind(p);
new person2();
```

而`call`则不会

```js
var p = {
  age: 18,
};
function Person() {
  console.log(this);
  console.log(this.age);
}

// call(p) 改变的是 Person 函数内部的 this
// 没有返回值
// person2 是 undefind，所以无法实例化
var person2 = Person.call(p);
new person2();

// 这样的方式和 Person.bind(p) 是一样的道理
Person.call(p);
new Person();
```

## 重写 bind() 方法
```js
Function.prototype.myBind = function (context) {
  var _this = this;
  // 将类数组转换成数字且从 1 开始截取
  var args = Array.prototype.slice.call(arguments, 1);

  return function () {
    var args2 = Array.prototype.slice.call(arguments);
    // 这里的 this 指向 widnow 所以外界要保存
    _this.apply(context, args.concat(args2));
  };
};

var p = {};
function Person(name, sex) {
  console.log(this);
  console.log(arguments);
}

Person.bind(p, "name1", "sex1")(); // {} ["name1", "sex1"]
Person.myBind(p, "name2", "sex2")(); // {} ["name2", "sex2"]
Person.myBind(p, "name3", "sex3")("name4", "sex4"); // {} ["name3", "sex3", "name4", "sex4"]
```
