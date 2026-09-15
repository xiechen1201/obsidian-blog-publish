---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/01 JavaScript 基础/08 Object 对象/00 Object 方法大集合/","dg-note-properties":{}}
---

---
---
`Object`构造函数的对象分为原型`prototype`和构造函数自身`constructor`方法：![原型方法](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1660099809687-95a996a1-672c-42f5-bdfb-51ecb388e549.png)

![静态方法](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1660099828282-352c589c-8bc3-494f-a9a7-f4103ab5eb2c.png)

## 🔢 创建对象
## 🔢 Object.create()
用于创建对象且给该对象指定一个原型，返回一个对象。

```js
var obj = Object.create({
  a: 3,
  b: 4,
});
console.log(obj);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1657781458761-84ed57e2-409c-43bf-8e1a-5cb0ccbbb02f.png)

## 🔢 定义属性的特性
## 🔢 Object.defineProperty()
用于定义单个对象属性的特性。

```js
var obj = Object.defineProperty({}, "test", {
  value: 1,
  writable: true,
  configurable: true,
  enumerable: true,
});

var value = 1;
var obj2 = Object.defineProperty({}, "test", {
  configurable: true,
  enumerable: true,
  set: function () {
    return value;
  },
  get: function (newVal) {
    value = newVal;
  }
});

console.log(obj2.test);
obj2.test = 2;
```

## 🔢 Object.defineProperties()
用于定义多个对象属性的特性。

```js
var value = 1;

var obj = Object.defineProperties({},
  {
    a: {
      configurable: true,
      enumerable: true,
      set: function () {
        return value;
      },
      get: function (newVal) {
        value = newVal;
      },
    },
    b: {
      value: 1,
      writable: true,
      configurable: true,
      enumerable: true,
    },
  }
);
```

更多细节：

[Object.defineProperty() / defineProperties()](https://www.yuque.com/xiechen/px9euv/hxm3f2)

## 🔢 读取属性的特性
## 🔢 Object.getOwnPropertyDescriptor()
用于获取对象单个属性的特性。

```js
var obj = Object.defineProperty({}, "a", {
  value: 1,
  writable: true,
  configurable: true,
  enumerable: true,
});

console.log(Object.getOwnPropertyDescriptor(obj, "a"));
// {value: 1, writable: true, enumerable: true, configurable: true}
```

## 🔢 Object.getOwnPropertyDescriptors()

ES2017 新增！

<br/>

用于获取对象多个属性的特性。

```js
var obj = Object.defineProperties({},
  {
    a: {
      value: 1,
      writable: true,
      configurable: true,
      enumerable: true,
    },
    b: {
      value: 2,
    },
  }
);

console.log(Object.getOwnPropertyDescriptors(obj));
// {value: 1, writable: true, enumerable: true, configurable: true}
// {value: 2, writable: false, enumerable: false, configurable: false}
```

## 🔢 合并对象
## 🔢 Object.assign()

ES2015 新增！

<br/>

用于将一个或多个源对象的所有可枚举属性复制到目标对象，返回修改后的目标对象。

```js
const target = { a: 1 };
const source = { b: 2, c: 3 };
const result = Object.assign(target, source);
console.log(result); // { a: 1, b: 2, c: 3 }
console.log(target === result); // true
```

## 🔢 对象遍历
## 🔢 Object.keys()
用于获取对象属性的`key`值（不包括对象的原型属性），返回`key`组成的数组。

```js
var obj = { a: 1, b: 2 };
Object.setPrototypeOf(obj, { c: 3, d: 4 });

console.log(Object.keys(obj)); // ["a", "b"]
```

## 🔢 Object.values()
用于获取对象属性的`value`值（不包括对象的原型属性），返回`value`组成的数组。

ES2017 新增！

<br/>

```js
var obj = { a: 1, b: 2 };
Object.setPrototypeOf(obj, { c: 3, d: 4 });

console.log(Object.values(obj)); // [1, 2]
```

## 🔢 Object.entries()
用于获取对象属性的`key`和`value`值（不包括对象的原型属性），返回`key`和`value`组成的二维数组。

ES2017 新增！

<br/>

```js
var obj = { a: 1, b: 2 };
Object.setPrototypeOf(obj, { c: 3, d: 4 });

console.log(Object.entries(obj)); // [["a", 1], ["b", 2]]
```

## 🔢 Object.fromEntries()

ES2019 新增！

<br/>

用于将二维数组转化为对象。

```js
const entries = [['a', 1], ['b', 2]];
const obj = Object.fromEntries(entries);
console.log(obj); // { a: 1, b: 2 }
```

## 🔢 操作对象的拓展性
## 🔢 Object.preventExtensions()
用于禁止对象拓展，调用方法后对象不可新增属性，但是可以读取、更改、删除，返回原对象。

```js
var obj = { a: 1, b: 2 };
Object.preventExtensions(obj);
console.log(Object.isExtensible(obj)); // false，不可拓展

obj.c = 3; // 新增属性无效
obj.a = 4; // 可以更改属性
delete obj.b; // 可以删除属性
console.log(obj.a); // 4，可以读取属性
console.log(obj); // {a: 4}
```

## 🔢 Object.isExtensible()
用于获取对象是否可拓展，返回布尔值。

```js
var obj = { a: 1, b: 2 };
console.log(Object.isExtensible(obj));
```

## 🔢 Object.seal()
用于封闭对象，封闭后的对象不可新增、删除，可以修改、读取，返回原对象。

```js
var obj = { a: 1, b: 2 };
Object.seal(obj);
console.log(Object.isExtensible(obj)); // false，不可拓展

obj.a = 4; // 可以修改
obj.c = 3; // 新增属性无效
delete obj.b; // 删除属性无效
console.log(obj.a); // 4，可以读取属性
console.log(obj); // {a: 4, b: 2}
```

## 🔢 Object.isSealed()
用来判断对象是否被封闭，返回布尔值。

```js
let obj = { name: "李四" };
let res = Object.seal(obj);
console.log(Object.isSealed(obj)); // true
console.log(obj === res); // true
```

## 🔢 Object.freeze()
用于冻结对象，冻结后的对象不可新增、修改、删除，可以读取，返回原对象。

```js
var obj = { a: 1, b: 2 };
Object.freeze(obj);
console.log(Object.isExtensible(obj)); // false，不可拓展

obj.a = 4; // 修改属性无效
obj.c = 3; // 新增属性无效
delete obj.b; // 删除属性无效
console.log(obj.a); // 1，可以读取属性
console.log(obj); // {a: 1, b: 2}
```

## 🔢 Object.isFrozen()
用来判断对象是否被冻结，返回布尔值。

```js
let obj = { name: "李四" };
let res = Object.freeze(obj);

console.log(Object.isFrozen(obj)); // true
console.log(obj === res); // true
```

## 🔢 获取对象本身属性
## 🔢 Object.getOwnPropertyNames()
用于获取对象非原型属性组成的数组。

```js
var obj = { a: 1, b: 2 };
Object.setPrototypeOf(obj, { c: 3, d: 4 });
console.log(Object.getOwnPropertyNames(obj)); // ['a', 'b']
```

## 🔢 Object.getOwnPropertySymbols()

ES2015 新增！

<br/>

用于获取对象所有自身的`Symbol`属性组成的数组。

```js
const obj = { [Symbol('a')]: 1 };
const symbols = Object.getOwnPropertySymbols(obj);
console.log(symbols); // [Symbol(a)]
```

## 🔢 Object.hasOwn()
用于查询对象是否具有某个属性，如果没有或者是原型上的属性则返回`false`。

```js
const object1 = {
  prop: 'exists',
};

console.log(Object.hasOwn(object1, 'prop')); // true
console.log(Object.hasOwn(object1, 'toString')); // false
```

建议使用此方法替代`Object.prototype.hasOwnProperty()`，因为它适用于使用`Object.create(null)` 创建的对象，以及重写了继承的 `hasOwnProperty()` 方法的对象。

## 🔢 操作原型
## 🔢 Object.getPrototypeOf()
用于获取对象的原型。

```js
var obj = { a: 1, b: 2 };
var proto = Object.getPrototypeOf(obj);
console.log(proto);

// 还可以通过属性直接获取
console.log(obj.__proto__);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1657778629445-5bc1688a-66b3-4c0e-a4e5-a58a09069559.png)

## 🔢 Object.setPrototypeOf()

ES2016 新增！

<br/>

用于给对象设置原型。

```js
var obj = { a: 1, b: 2 };
Object.setPrototypeOf(obj, { c: 3, d: 4 });
console.log(obj);

// 还可以直接设置对象的原型
obj.__proto__ = { c: 3, d: 4 };
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1657778439005-01d82f2e-6e84-47f1-aac3-4536dcb2a906.png)

## 🔢 数据对比
## 🔢 Object.is()

ES2015 新增！

<br/>

用于判断两个值是否是相同的值。与`===`不同的是，`Object.is()`正确地处理了`NaN`和`-0`。

```js
console.log(Object.is(NaN, NaN)); // true
console.log(Object.is(-0, 0)); // false
```
