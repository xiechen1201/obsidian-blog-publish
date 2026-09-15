---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/07 原型、原型链、继承/03 原型与原型链深入、对象继承、call apply/","dg-note-properties":{}}
---

---
---
## 🔢 ![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1658203019103-4473db85-e60f-40af-8d3d-e3b455a3353a.png)
## 🔢 原型
`__proto__`保存的「原型」指向`function.prototype`

```js
function Car() {}

var car = new Car();
console.log(Car.prototype);
console.log(car);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650953523329-5a567eac-a79a-48c2-a640-4d2d9d90d864.png)

结合上面的图我们能看到`car.__proto__`的原型是`Car.prototype`，`Car.prototype.__proto__`的原型是`Object.prototype`

所以：所有的对象都有自己的原型，包括原型本身！

![画板](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650953861745-c9ad0f6c-e8e7-47f4-895e-c5fec2ad2b8a.jpeg)

## 🔢 函数的原型
在 JS 中函数也属于对象，但是函数的原型和普通对象的原型是不一样的：

```js
function Person(name) {
    console.log(name);
}
console.log(Person.__proto__); // ƒ () { [native code] }
console.log(Person.__proto__ === Function.prototype); // true
```

<br/>warning
⚠️ 注意

`Person.__proto__`实际上指向的是`Function.prototype`，其实就是函数的原型对象。所有函数（包括`Person`）本质上是由`Function`构造器创建的，所以`Person`的原型指向了`Function.prototype`。

<br/>

因此，`Person`属于`Function`的实例：

```js
console.log(Person instanceof Function); // true
```

`Function`是一个特殊的函数，且它的`__proto__`指向自己：

```js
console.log(Function.__proto__ === Function.prototype); // true
```

而`Function.prototype`的原型才会指向`Object.prototype`:

```js
Function.prototype.__proto__ === Object.prototype; // true
```

`Function`对象和普通对象原型链对比：

```js
// 普通对象
obj.__proto__ → Object.prototype → null
// Function 对象
Person.__proto__ → Function.prototype → Object.prototype → null
```

## 🔢 原型链
其实上面👆的情况就是「原型链」。

「原型链」就是去「原型对象」里一层一层寻找相应的属性的这样的继承属性链就是「原型链」（没有的属性先到我自己的实例上寻找，实例上找不到就去原型对象上寻找，如果原型对象上也没有就继续到原型对象的原型对象去寻找）。

```js
function Professor() {}
Professor.prototype.tSkill = "JAVA";
var professor = new Professor();

function Teacher() {
  this.mSkill = "JS/JQ";
}
Teacher.prototype = professor;
var teacher = new Teacher();

function Student() {
  this.pSkill = "HTML/CSS";
}
Student.prototype = teacher;
var student = new Student();

console.log(student);
console.log(student.mSkill);
console.log(student.tSkill);
```

![画板](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1709693009288-52aef816-2004-4979-a308-b92b6f4c0c03.jpeg)

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650954168999-32d084f9-f993-483b-a881-6cffa399b27b.png)

`student`实例对象是完全可以访问到`mSkill`和`tSkill`的，他会沿着原型链条一直寻找，直到顶端。

## 🔢 原型链的顶端
原型链的顶端是`Object.prototype`:

```js
function Professor(){}
console.log(Professor.prototype);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650954439287-3b8b660e-b349-4c8b-8627-642da8f89124.png)

原型的原型是由系统自带的构造函数`function Object(){}`构造出来的:

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650955605885-4c207862-7f63-431d-84e1-42656bb8a6dd.png)

## 🔢 实例对象操作原型链上的属性
```js
function Professor() {}
Professor.prototype.tSkill = "JAVA";
var professor = new Professor();

function Teacher() {
  this.mSkill = "JS/JQ";
  this.students = 500;
  this.success = {
    alibaba: 28,
    tencent: 30,
  };
}
Teacher.prototype = professor;
var teacher = new Teacher();

function Student() {
  this.pSkill = "HTML/CSS";
}
Student.prototype = teacher;
var student = new Student();
```

现在先尝试去修改`Teacher`下的`success`对象

```js
student.success.baidu = 100;

console.log(student, teacher);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650954723090-97a912ed-f31a-43f8-81e4-92641e1aa003.png)

现在我们能看到`student`实例对象是完全可以更改原型链上引用数据的。

那我们再来看更改一下原始数据：

```js
student.students++;

console.log(student, teacher);

// 当执行到 student.students++ 时，
// 实例对象发现自己没有 students 属性就会到原型上找
// 然后就会执行 this.student = teacher.students++
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650954846219-5764fcee-1256-4c62-af31-b82d5356c7cf.png)

我们可以看到当用实例对象`student`去更改原型上的原始数据，`students`属性却新增到实例对象本身上！！！

由此总结：

- 实例对象更改原型链上的「引用数值」会更改原型链上的属性；
- 实例对象更改原型链上的「原始数值」会新增到自己的实例上；

## 🔢 this 指向
当构造函数和构造函数原型上有相同的属性时，谁使用`this`访问，`this`就指向谁！！！

```js
function Car() {
  this.brand = "Benz";
}
Car.prototype = {
  brand: "Mazsa",
  intor: function () {
    console.log("我是" + this.brand + "车");
  },
};
var car = new Car();

// car 实例对象访问 intor() 函数
car.intor(); // 我是Benz车

// prototype 访问 intor() 函数
car.prototype.intor(); // 我是Mazsa车
```

## 🔢 Object.create()

`Object`构造函数静态方法。

<br/>

<u>实例化对象的另外一种写法就是使用</u>`<u>Object.create()</u>`<u>方法，该方法接受一个对象或者</u>`<u>null</u>`<u>做为参数。</u>

<u></u>

```js
function Obj() {}
Obj.prototype.num = 1;

// 两种方法构建出的对象一模一样
var obj1 = Object.create(Obj.prototype);
var obj2 = new Obj();

console.log(obj1);
console.log(obj2);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650955766190-ffacf58c-7b39-43b6-8e17-00ab5385fb0a.png)

从外面来看使用`new`和`Object.create()`创建出来的对象没有任何的区别。

`Object.create()`的好处只是可以给一个对象自定义原型。

指定原型是一个对象：

```js
var test = {
  num: 2,
};
var obj3 = Object.create(test);

console.log(obj3);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650955910443-6965827d-23a4-4c94-9814-981c1ae89c0b.png)

指定原型是`null`：

```js
var obj1 = Object.create(null);
console.log(obj1);
obj1.num = 1;

var obj2 = Object.create(obj1);
console.log(obj2);
console.log(obj2.num);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/07%20%E5%8E%9F%E5%9E%8B%E3%80%81%E5%8E%9F%E5%9E%8B%E9%93%BE%E3%80%81%E7%BB%A7%E6%89%BF/_assets/1650955993103-753bda90-333b-4c93-a17e-3d67546d7601.png)

当一个对象的原型被指定为`null`的时候，该对象无法调用任何的方法。

```js
var obj = Object.create(null);
obj.num = 1;
obj.toString(); // error，not a function
```

这是因为`obj`被创建的时候原型指定为`null`，所以`obj`就没有原型，所以`obj`是无法根据原型链的规则往上寻找到`Object.prototype`的，因此也就无法使用任何的方法。

还因为`obj`没有原型，所以给它赋值`obj.__proto__`是无法使用的。

```js
var obj = Object.create(null);
var obj1 = { count: 1 };

obj.__proto__ = obj1; // 这里的赋值相当于赋值了一个普通的属性
console.log(obj.count); // 所以无法直接访问 count
```

## 🔢 new 的过程
- 创建`this`对象
- `this`对象保存构造函数的`prototype`
- `this`对象初始化属性和方法
- 返回`this`对象

## 🔢 包装类的方法
```js
var num = new Number(1);
console.log(num.toString()); // "1"

var bool = new Boolean("true");
console.log(bool.toString()) // "true"
```

因为包装类也是实例化对象，实例化对象是可以访问到`prototype`的，所以包装类是完全可以调用自身的方法！！！

而`undefind`和`null`是无法通过包装类进行包装的，所以它两也就无法调用方法！！！

```js
console.log(undefined.toString()); // error
console.log(null.toString()); // error
```
