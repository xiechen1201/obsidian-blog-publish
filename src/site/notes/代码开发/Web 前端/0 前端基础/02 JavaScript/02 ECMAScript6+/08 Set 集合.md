---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/02 ECMAScript6+/08 Set 集合/","dg-note-properties":{}}
---

---
---
一直以来 JS 只能使用数组和对象来保存多数据，缺乏像其他语言那样拥有丰富的集合类型。因此 ES6 新增了两种集合类型：`Set`和`Map`，用于在不同的场景下发挥作用。

其中，`Set`对象用于存放不重复的数据。

## 🔢 创建 Set 集合
要创建`Set`对象实例化`Set()`构造函数即可：

```js
const s = new Set(); // 创建一个空的 Set 集合
console.log(s); // Set(0) {}
```

`Set()`构造函数接收一个可迭代对象：

```js
// 创建一个具有初始化内容的 set 集合
// 内容来自于可迭代对象每一次迭代的结果
const s1 = new Set([1, 2, 3, 4, 5]);
console.log(s1); // Set(5) { 1, 2, 3, 4, 5 }
```

`Set`对象可以实现自动去重：

```js
const s2 = new Set([1, 1, 2, 2, 3, 3, 4, 4, 5, 5]);
console.log(s2); // Set(5) { 1, 2, 3, 4, 5 }
```

如果传递给`Set()`构造函数的是字符串，也会转换为`String`对象，`String`对象也是可迭代对象：

```js
const s3 = new Set('hello');
console.log(s3); // Set(4) { 'h', 'e', 'l', 'o' }
```

## 🔢 Set 集合的方法
1、`.add(value)`向集合的末尾添加元素，如果数据已存在则不进行任何的操作。

如何判断数据是否重复？

底层使用的是`Object.is()`方法来判断两个数据是否相同，但是针对`+0`和`-0`则认为是相等的，针对对象判断的是对象的内存地址是否一样。

<br/>

```js
const s = new Set();
s.add(1);
s.add(2);
console.log(s); // Set(2) { 1, 2 }
```

2、`.has(value)`判断集合中是否存在某个元素，返回`boolean`值。

```js
const s = new Set();
s.add(1);
s.add(2);

console.log(s.has(1)); // true
console.log(s.has(3)); // false
```

3、`.delete(value)`删除集合中的匹配元素，返回`boolean`，表示是否删除成功。

```js
const s = new Set();
s.add(1);
s.add(2);

s.delete(1);

console.log(s); // Set(1) { 2 }
```

4、`.clear()`清空集合中所有的元素。

```js
const s = new Set();
s.add(1);
s.add(2);

s.clear();

console.log(s); // Set(0) {}
```

5、`.size`属性，用于获取`Set`集合中元素的数量，是一个只读属性。

```js
const s = new Set();
s.add(1);
s.add(2);
console.log(s.size); // 2
```

## 🔢 Set 集合和数组转换
`Set`去重并转换为数组：

```js
let arr = [1, 1, 2, 2, 3, 3, 4, 4, 5, 5];
let s = new Set(arr);

console.log(s); // Set(5) { 1, 2, 3, 4, 5 }

// set 本身也是一个可迭代对象，每次迭代的结果就是每一项的值
arr = [...s];
console.log(arr); // [ 1, 2, 3, 4, 5 ]
```

`Set`去重并转换为字符串：

```js
let str = 'hello';
str = [...new Set(str)].join('');
console.log(str); // helo
```

## 🔢 Set 集合遍历
1、因为`Set`集合也是可迭代对象，所以可以使用`for...of...`进行迭代。

```js
const s = new Set([1, 2, 3, 4, 5]);

for (const element of s) {
  console.log(element);
}

/*
  1
  2
  3
  4
  5
*/
```

2、`.forEach(callback)`方法，使用方式同数组的`Array.prototype.forEach()`方法。

```js
const s = new Set([1, 2, 3, 4, 5]);

s.forEach((value, key, set) => {
  console.log(value, key, set);
});

/*
  1 1 Set(5) { 1, 2, 3, 4, 5 }
  2 2 Set(5) { 1, 2, 3, 4, 5 }
  3 3 Set(5) { 1, 2, 3, 4, 5 }
  4 4 Set(5) { 1, 2, 3, 4, 5 }
  5 5 Set(5) { 1, 2, 3, 4, 5 }
*/
```

<br/>warning
⚠️ 注意

`Set`集合不存在下标！下标是数组的专有属性。

这里回调函数的第二个参数不是`index`下标，而是为了保持和`Array.prototype.forEach()`格式统一，提供了第二个参数。但是第二个参数和第一个参数内容是一致的，均为`Set`集合的元素。

<br/>

3、使用`.keys()`、`.values()`、`.entries()`方法。这三个方法都返回的是迭代器对象！

```js
const s = new Set([1, 2, 3, 4, 5]);

console.log(s.keys());
console.log(s.values());
console.log(s.entries());

/*
  [Set Iterator] { 1, 2, 3, 4, 5 }
  [Set Iterator] { 1, 2, 3, 4, 5 }
  [Set Entries] { [ 1, 1 ], [ 2, 2 ], [ 3, 3 ], [ 4, 4 ], [ 5, 5 ] }
*/
```

需要使用`for...of...`进行迭代：

```js
for (const element of s.keys()) {
  console.log(element)
}

/*
  1
  2
  3
  4
  5
*/

for (const element of s.values()) {
  console.log(element)
}

/*
  1
  2
  3
  4
  5
*/

for (const element of s.entries()) {
  console.log(element)
}

/*
  [ 1, 1 ]
  [ 2, 2 ]
  [ 3, 3 ]
  [ 4, 4 ]
  [ 5, 5 ]
*/
```

## 🔢 模拟 Set 集合
```js
class MySet {
  constructor(iterator = []) {
    // 验证是否是可迭代的对象
    if (typeof iterator[Symbol.iterator] !== 'function') {
      throw new Error(`${iterator}不是一个可迭代的对象`);
    }

    // 声明一个数组用于存储数据
    this._datas = [];

    for (const element of iterator) {
      this.add(element);
    }
  }

  get size(){
    return this._datas.length;
  }

  add(value) {
    // 如果数组中不存在这个数据，则执行 push
    if (!this.has(value)) {
      this._datas.push(value);
    }
  }

  delete(value) {
    for (let index = 0; index < this._datas.length; index++) {
      const element = this._datas[index];

      // 判断两个数据是否相等
      if (this.isEqual(value, element)) {
        this._datas.splice(index, 1);
        // 删除成功后返回 true
        return true;
      }
    }
    // 否则返回 false
    return false;
  }

  has(value) {
    for (const element of this._datas) {
      if (this.isEqual(value, element)) {
        return true;
      }
    }
    return false;
  }

  clear() {
    this._datas = [];
  }

  isEqual(data1, data2) {
    // 如果两个数都为 0 无论 +0 还是 -0，则认为它们是相等的
    if (data1 === 0 && data2 === 0) {
      return true;
    }
    // 否则使用 Object.is() 方法
    return Object.is(data1, data2);
  }

  // 使用生成器写一个迭代器
  *[Symbol.iterator]() {
    for (const element of this._datas) {
      yield element;
    }
  }

  // 模拟 forEach 方法
  forEach(callback) {
    for (const element of this._datas) {
      callback(element, element, this);
    }
  }
}
```

实例化`MySet()`构造函数：

```js
const s = new MySet([1, 2, 3, 1, 2, 3]);

console.log(s); // MySet { _datas: [ 1, 2, 3 ] }

s.add("a");
s.add("b");

console.log(s.delete(3)); // true

for (const element of s) {
  console.log(element);
}

/*
  1
  2
  a
  b
*/

s.forEach((el, index, set) => console.log(el, index, set));

/*
  1 1 MySet { _datas: [ 1, 2, 'a', 'b' ] }
  2 2 MySet { _datas: [ 1, 2, 'a', 'b' ] }
  a a MySet { _datas: [ 1, 2, 'a', 'b' ] }
  b b MySet { _datas: [ 1, 2, 'a', 'b' ] }
*/
```
