---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/02 ECMAScript6+/10 Map 集合/","dg-note-properties":{}}
---

`Map`集合是专门用于存储多个键值对的集合。

在`Map`出现之前使用的是对象来存储键值对，使用对象来存储有个特点：键名不能重复。

```js
let x = { id: 1 },
		y = { id: 2 };
let m = {};

m[x] = "foo";
m[y] = "bar";
console.log(m); // {[object Object]: 'bar'}
```

另外还存在以下几个问题：

1、键名只能是字符串或者`Symbol`，不能是其他的类型。`Map`集合的键可以是任意的类型。

2、获取数据的数量不方便。

3、键名很容易和原型的键产生冲突。

## 创建 Map 集合
要创建`Map`集合实例化`Map()`构造函数即可：

```js
const m = new Map(); // 创建一个空的 Map 集合
console.log(m); // Map(0) {}
```

`Map()`构造函数还接收一个可迭代对象：

```js
let m = new Map([
  ['name', '张三'],
  ['age', 18],
  ['gender', '男']
]);
console.log(m); // Map(3) { 'name' => '张三', 'age' => 18, 'gender' => '男' }
```

以上代码，创建一个具有初始内容的`Map`集合，初始内容来自于可迭代对象每一次迭代的结果，它要求每次迭代的结果必须是一个长度为 2 的数组，第一项表示键，第二项表示值。

<br/>

`Map`集合的键可以是任意的数据类型：

```js
const map = new Map();

map.set(-0, "123");
map.set(true, "456");
map.set("true", "678");
map.set(undefined, "10");
map.set(null, 2);
map.set(NaN, 20);

console.log(map.get(+0)); // 123
console.log(map.get(true)); // 456
console.log(map.get(undefined)); // 10
console.log(map.get(null)); // 2
console.log(map.get(NaN)); // 20
console.log(map.size); // 6
```

## Map 集合的方法
1、`.set(key, value)`向`Map`集合中添加键值对，键和值可以是任意的类型。如何`Map`中不存在要添加键则进行添加，如果已经存在键则进行修改键对应的值。

```js
let m = new Map();

m.set('name', '张三');
m.set({}, '李四');
m.set('name', '王五');

console.log(m); // Map(2) { 'name' => '王五', {} => '李四' }
```

2、`.get(key)`获取指定键对应的值。

```js
let m = new Map();

m.set('name', '张三');
m.set({}, '李四');

console.log(m.get('name')); // 张三
console.log(m.get('abc')); // undefined
```

3、`.has(key)`判断`Map`集合中是否具有某个键，返回`boolean`值。

```js
let m = new Map();

m.set('name', '张三');
m.set({}, '李四');

console.log(m.has('name')); // true
console.log(m.has('abc')); // flase
```

4、`.delete(value)`删除集合中的匹配元素，返回`boolean`，表示是否删除成功。

```js
let m = new Map();

m.set('name', '张三');
m.set({}, '李四');

console.log(m.delete('name')); // true
console.log(m); // Map(1) { {} => '李四' }
```

5、`.clear()`清空集合中所有的元素。

```js
let m = new Map();

m.set('name', '张三');
m.set({}, '李四');

m.clear()
console.log(m); // Map(0) {}
```

6、`.size`属性，用于获取`Map`集合中元素的数量，是一个只读属性。

```js
let m = new Map();

m.set('name', '张三');
m.set({}, '李四');

console.log(m.size); // 2
```

## Map 集合转化为数组
```js
let m = new Map([
  ['name', '张三'],
  ['age', 18],
  ['gender', '男']
]);
console.log(m); // Map(3) { 'name' => '张三', 'age' => 18, 'gender' => '男' }

// 使用展开运算符即可
let arr = [...m];
console.log(arr); // [ [ 'name', '张三' ], [ 'age', 18 ], [ 'gender', '男' ] ]
```

## Map 集合遍历
1、因为`Map`集合也是可迭代对象，所以可以使用`for...of...`进行迭代。

```js
let m = new Map([
  ['name', '张三'],
  ['age', 18],
  ['gender', '男']
]);

for (const element of m) {
  console.log(element);
}

/*
  [ 'name', '张三' ]
  [ 'age', 18 ]
  [ 'gender', '男' ]
*/
```

2、`.forEach(callback)`方法，使用方式同数组的`Array.prototype.forEach()`方法。

```js
m.forEach((value, key, map) => {
  console.log(key, value, map);
});

/*
  name 张三 Map(3) { 'name' => '张三', 'age' => 18, 'gender' => '男' }
  age 18 Map(3) { 'name' => '张三', 'age' => 18, 'gender' => '男' }
  gender 男 Map(3) { 'name' => '张三', 'age' => 18, 'gender' => '男' }
*/
```

> [!warning]
>
> `map.forEach()`和`set.forEach()`函数一样，`callback()`回调函数的第二个参数并不是`index`下标，而是对象的`key`。


3、使用`.keys()`、`.values()`、`.entries()`方法。这三个方法都返回的是迭代器对象！

```js
console.log(m.keys()); // [Map Iterator] { 'name', 'age', 'gender' }
console.log(m.values()); // [Map Iterator] { '张三', 18, '男' }
console.log(m.entries()); // [Map Entries] { [ 'name', '张三' ], [ 'age', 18 ], [ 'gender', '男' ] }
```

需要使用`for...of...`进行迭代：

```js
for (const element of m.keys()) {
   console.log(element)
}

/*
  name
  age
  gender
*/

for (const element of m.values()) {
    console.log(element)
}

/*
  张三
  18
  男
*/

for (const element of m.entries()) {
    console.log(element)
}

/*
  [ 'name', '张三' ]
  [ 'age', 18 ]
  [ 'gender', '男' ]
*/
```

## 模拟 Map 集合
```js
class MyMap {
  constructor(iterator = []) {
    // 判断实例化的时候传递的参数是不是一个可迭代对象
    if (typeof iterator[Symbol.iterator] !== 'function') {
      throw new Error(`${iterator}不是一个可迭代的对象`);
    }

    // 用于保存数据
    this._datas = [];

    // 迭代数据，例如 iterator = [['name', '张三'], ['age', 18]]
    for (const element of iterator) {
      // element 也必须是一个可迭代对象
      if (typeof element[Symbol.iterator] !== 'function') {
        throw new Error(`${element}不是一个可迭代的对象`);
      }

      // 得到一个迭代对象
      const iterator = element[Symbol.iterator]();
      // 得到 'name'
      const key = iterator.next().value;
      // 得到 '张三‘
      const value = iterator.next().value;

      this.set(key, value);
    }
  }

  get(key) {
    const obj = this._getObj(key);
    return obj ? obj.value : undefined;
  }

  set(key, value) {
    const obj = this._getObj(key);

    if (obj) {
      // 如果存在键，则进行数据修改
      obj.value = value;
    } else {
      // 否则就添加进去
      this._datas.push({ key, value });
    }
  }

  has(key) {
    const item = this._getObj(key);
    return item !== undefined;
  }

  delete(key) {
    for (let index = 0; index < this._datas.length; index++) {
      const element = this._datas[index];

      if (this.isEqual(element.key, key)) {
        // 如果删除成功则返回 true
        this._datas.splice(index, 1);
        return true;
      }
    }
    // 否则返回 false
    return false;
  }

  clear() {
    this._datas = [];
  }

  isEqual(data1, data2) {
    if (data1 === 0 && data2 === 0) {
      return true;
    }
    return Object.is(data1, data2);
  }

  // 根据 key 从内部数组找到对应的数组项
  _getObj(key) {
    for (const element of this._datas) {
      if (this.isEqual(element.key, key)) {
        return element;
      }
    }
    return undefined;
  }

  // 部署一个生成器函数
  *[Symbol.iterator]() {
    for (const element of this._datas) {
      yield [element.key, element.value];
    }
  }

  // 部署一个 forEach 函数
  forEach(callback) {
    for (const element of this._datas) {
      // 把 value 和 key 传递给回调函数
      callback(element.value, element.key, this);
    }
  }
}
```

实例化`MyMap()`构造函数：

```js
const m = new MyMap([['name', 'zhangsan']]);
m.set('age', 18);
console.log(m);

/*
MyMap {
  _datas: [ { key: 'name', value: 'zhangsan' }, { key: 'age', value: 18 } ]
}
*/

for (const element of m) {
  console.log(element);
}

/*
  [ 'name', 'zhangsan' ]
  [ 'age', 18 ]
*/

m.forEach((value, key, map) => {
  console.log(value, key, map);
});

/*
  zhangsan name MyMap {
    _datas: [ { key: 'name', value: 'zhangsan' }, { key: 'age', value: 18 } ]
  }
  18 age MyMap {
    _datas: [ { key: 'name', value: 'zhangsan' }, { key: 'age', value: 18 } ]
  }
*/
```
