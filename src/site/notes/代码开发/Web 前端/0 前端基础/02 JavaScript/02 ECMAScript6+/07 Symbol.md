---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/02 ECMAScript6+/07 Symbol/","dg-note-properties":{}}
---

---
---
`Symbol`（符号）是 ES6 新增的原始数据类型，表示独一无二的值，它是通过`Symbol(描述)`来创建的。

`ES5`的原始数据类型：`string`、`number`、`boolean`、`null`、`undefined`。

所以自 ES6 后有 6 个原始数据类型。

<br/>

`ES5`的对象属性名都是字符串，这容易造成属性名的冲突。比如你使用了一个他人提供的对象，但又想为这个对象添加新的方法，新方法的名字就有可能与现有方法产生名字冲突。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。

`Symbol()`设计的初衷，是为了给对象设置私有属性:

```js
const hero = {
  attack: 30,
  hp: 300,
  defence: 10,
  gongji() {
    const dmg = this.attack + this.getRandmom(1, 100);
    console.log(dmg);
  },
  getRandmom(min, max) {
    return Math.random() * (max - min) + min;
  }
};
```

如果我不想让`hero`对象外部调用`getRandmom()`方法可以尝试把`getRandmom()`方法写在`gongji()`方法的内部：

```js
const hero = {
  // ...
  gongji() {
    function getRandmom(min, max) {
      return Math.random() * (max - min) + min;
    }

    const dmg = this.attack + getRandmom(1, 100);
    console.log(dmg);
  }
};
```

这样虽然解决了外部不能调用`getRandmom()`方法的情况，但是这样会造成每次调用`gongji()`方法都会创建一个`getRandmom()`方法，并且`hero`对象内的其他成员也无法共享！

所以 ES6 使用`Symbol()`来创建私有属性。

## 🔢 普通符号
1、`Symbol()`的基础使用：

```js
console.log(Symbol()); // Symbol()
```

2、`Symbol()`方法可以传递一个参数，表示对`Symbol`的描述：

```js
let s1 = Symbol("s1");
console.log(s1); // Symbol(s1)
```

3、每次调用`Symbol()`都会返回一个全新的 symbol 符号，即使传入的参数相同，也会返回新的 symbol 符号：

```js
let s1 = Symbol("sss");
let s2 = Symbol("sss");

console.log(s1 === s2); // false
```

`Symbol()`的描述只是起到一个调试的作用，调式的时候可以方便知道这个符号的作用。

<br/>

可以简单的把`Symbol()`返回的内容理解为一个不可能重复的随机数，仅方便自己理解，但`Symbol()`返回的并不是随机数！！！

<br/>

4、`Symbol()`的值是字符串，标识名会转换为字符串：

```js
let obj = { a: 1 };
let s1 = Symbol(obj);

console.log(s1); // Symbol([object Object])
```

符号可以当作对象的属性名存在，这种属性被称为符号属性。

```js
const syb = Symbol('testName');

const obj = {
  a: '1',
  b: '2',
  [syb]: '3' // 符号属性
};
console.log(obj);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721119878376-1edf92a2-6d07-45b4-b2ef-0bdf27badbb9.png)

开发者可以使用`Symbol()`让这些属性无法通过常规的方式被外界访问到：

```js
const hero = (function () {
  const symbol = Symbol('getRandmom');

  return {
    attack: 30,
    hp: 300,
    defence: 10,
    gongji() {
      const dmg = this.attack + this[symbol](1, 100);
      console.log(dmg);
    },
    [symbol](min, max) {
      return Math.random() * (max - min) + min;
    }
  };
})();

console.log(hero); // { attack: 30, hp: 300, defence: 10, gongji: [Function: gongji], [Symbol(getRandmom)]: [Function: [getRandmom]] }
hero.gongji(); // 80.34319887536014
```

既然说了不能通过常规的方式获取到，那么肯定是有不常规的方式！

在 ES6 中为了方便获取到`Symbol()`的属性提供了`Object.getOwnPropertySymbols(target)`方法。

```js
let obj = {};
let sym1 = Symbol('foo');
let sym2 = Symbol('bar');

obj[sym1] = 'value1';
obj[sym2] = 'value2';

let symbols = Object.getOwnPropertySymbols(obj);

console.log(symbols); // 输出 [Symbol(foo), Symbol(bar)]
console.log(obj[symbols[0]]); // 输出 'value1'
console.log(obj[symbols[1]]); // 输出 'value2'
```

所以我可以利用`Object.getOwnPropertySymbols(target)`方法获取到`hero`对象内的 symbol 属性：

```js
const symbols = Object.getOwnPropertySymbols(hero);
console.log(symbols); // [Symbol(getRandmom)]

let num = hero[symbols[0]](1, 100);
console.log(num); // 93.25415049820366
```

5、符号属性是不能进行枚举的，因此在`for...in...`中是无法读取到符号属性的，即使使用`Object.keys()`方法也无法读取到符号属性。

```js
const symbol = Symbol();

const obj = {
  a: 1,
  b: 2,
  [symbol]: 3
};

for (const key in obj) {
  console.log(key); // a b
}
```

所以 ES6 提供了`Object.getOwnPropertySymbols(target)`方法！

6、`Symbol()`无法被隐式转化，所以不能进行数学的运算、字符串的拼接或者其他隐式转化的场景。

但是符号是可以显式转化为字符串的，通过`String()`构造函数转化即可。

```js
const symbol = Symbol();
console.log(symbol + 1); // ❌TypeError: Cannot convert a Symbol value to a number
console.log(symbol + 'abc'); // ❌TypeError: Cannot convert a Symbol value to a string
```

```js
const symbol = Symbol();
console.log(String(symbol) + 'abc'); // Symbol()abc
```

## 🔢 共享符号
共享符号的意思是根据某个符号描述来得到同一个符号。

```js
const s1 = Symbol('foo');

const obj1 = {
  a: 1,
  b: 2,
  [s1]: 3
};

const obj2 = {
  a: 'a',
  b: 'b',
  [s1]: 'c'
};

console.log(obj1, obj2); // { a: 1, b: 2, [Symbol(foo)]: 3 } { a: 'a', b: 'b', [Symbol(foo)]: 'c' }
```

如果是两个对象使用同一个变量是没有任何的问题的，因为变量指向同一个符号。

如果符号在其他的地方，那么对象该如何获取这个符号呢？

那么可以使用`Symbol.for('描述')`这个静态方法来获取共享的符号。

```js
const s1 = Symbol.for('foo');
const s2 = Symbol.for('foo');

console.log(s1 === s2); // true

const obj = {
  [Symbol.for('foo')]: 3
};

console.log(obj); // { [Symbol(foo)]: 3 }
console.log(obj[Symbol.for('foo')]); // 3
```

`Symbol.for()`与`Symbol()`这两种写法，都会生成新的`Symbol`。它们的区别是前者会被登记在全局环境中供搜索，后者不会：

```js
let s1 = Symbol("foo");
let s2 = Symbol("foo");
let s3 = Symbol.for("foo");
let s4 = Symbol.for("foo");

console.log(s1 === s2); // false
console.log(s3 === s4); // true
console.log(s1 === s4); // false
```

`Symbol()`还有一个静态方法`Symbol.keyFor(symbol)`，用于查找已全局登记的 symbol 的描述：

```js
let s = Symbol("foo");
let s1 = Symbol.for("foo");

console.log(Symbol.keyFor(s)); // undefined
console.log(Symbol.keyFor(s1)); // foo
```

然后就可以根据得到的描述去使用已注册的符号：

```js
let s = Symbol.for('foo');
let d = Symbol.keyFor(s);
let obj = {
  [Symbol.for(d)]: 3
};

console.log(obj); // {Symbol(foo): 3}
```

⚠️ 注意

该方法只能获取`Symbol.for()`返回的 symbol。

<br/>

## 🔢 知名符号
知名符号是一些具有特殊含义的共享符号，通过`Symbol`的静态属性得到。

ES6 延续了 ES5 的思想，尽量为这个语言减少魔法，暴露出内部的实现，因此 ES6 用知名符号暴露某些场景的内部实现。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1721181319717-e85300fd-62ee-4d02-9d82-02dd01b8aec1.png)

1、`Symbol.hasInstance`

该属性用于定义构造函数的静态成员，它将影响`instanceof`的判定。

```js
obj instanceof A
// 等效于
A[Symbol.hasInstance](obj);
```

一般情况下，`obj`一定是`A`的实例对象：

```js
function A() {}
const obj = new A();

console.log(obj instanceof A); // true
console.log(A[Symbol.hasInstance](obj)); // true
```

我可以使用`Symbol.hasInstance`来更改这一默认的行为：

```js
function A() {}
const obj = new A();

Object.defineProperty(A, Symbol.hasInstance, {
  value: function (obj) {
    console.log(obj);
    return false;
  }
});

obj instanceof A; // flase
```

2、`Symbol.isConcatSpreadable`

该符号会影响数组的`.concat()`方法。

`.concat()`内部发现参数有下标，并且具有`.length`属性，那么就会当作数组进行拆分。

```js
const arr = [1, 2, 3];
let result = arr.concat(4, [5, 6]);

console.log(result); // [1, 2, 3, 4, 5, 6]
```

我可以使用`Symbol.isConcatSpreadable`来更改这一行为：

```js
const arr = [1, 2, 3];
arr[Symbol.isConcatSpreadable] = false;
let result = arr.concat(4, [5, 6]);

console.log(result); // [[1, 2, 3], 4, 5, 6]
```

3、`Symbol.toPrimitive`

该符号会影响类型转换的结果。

一般情况下，如果对象参与运算那么对象会调用`.valueOf()`方法：

```js
const obj = {
  a: 1,
  b: 2
};
// obj 首先会调用 .valueOf() 方法
// 如果返回值不是原始值，那么会继续调用 .toString() 方法得到 [object Object]
console.log(obj + 10); // [object Object]10
```

我可以使用`Symbol.toPrimitive`来更改这一行为：

```js
const obj = {
  a: 1,
  b: 2
};
obj[Symbol.toPrimitive] = function () {
  return 100;
};
console.log(obj * 10); // 1000
```

4、`Symbol.toStringTag`

该符号会影响`Object.prototype.toString()`的返回结果。

一般情况下，对象调用`.toString()`方法会返回`[object Object]`字符串：

```js
const obj = {
  a: 1,
  b: 2,
  c: 3
};
console.log(obj.toString()); // [object Object]
```

我可以利用`Symbol.toStringTag`来更改这一行为：

```js
const obj = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.toStringTag]: 'Person'
};
console.log(obj.toString()); // [object Person]
```
