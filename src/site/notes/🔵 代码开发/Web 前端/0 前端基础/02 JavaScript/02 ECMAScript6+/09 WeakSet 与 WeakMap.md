---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/0 前端基础/02 JavaScript/02 ECMAScript6+/09 WeakSet 与 WeakMap/","dg-note-properties":{}}
---

---
---
## 🔢 对象引用
如何理解对象的引用呢？

```js
let obj1 = {
  a: 1
};

let obj2 = obj1;

const m = new Map();
m.set(obj1, 1);
```

思考下，以上代码中`{a: 1}`总共被引用了几次？

答案是 3 次，分别是`obj1`、`obj2`和`m`。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1687673207292-95c3d431-ac38-424c-9e17-396510bdca50.png)

需要注意的是`m.set(obj1)`和`obj1`并不是一个东西，只是`map`的键值指向了`{a: 1}`，它们都是指针的关系！！！

![画板](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1687673566894-8b8d1004-59bf-4d17-a795-31718f9f8366.jpeg)

如果我们把`obj1`和`obj2`的指向断开，并不会影响到`m`的数据：

```js
let obj1 = {
  a: 1
};

let obj2 = obj1;

const m = new Map();
m.set(obj1, 1);

// 只是把链接 { a: 1 } 的线剪断了，而 { a: 1 } 依然存在于堆内存中
obj1 = null;
obj2 = null;

// 不能通过 m.get(obj1) 来访问，因为这个时候 obj1 已经不存在了。
// 但是 { a: 1 } 依然是 m 的键名
console.log(m);
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1687673330437-89da96a2-8585-4d61-8433-3a012555ae71.png)

我们都知道 JS 中存在变量的垃圾回收机制，那么结合本案例，`{a: 1}`会在什么时候被回收呢？

实际上，只有当`{a: 1}`不被任何变量引用的时候才会被垃圾回收。而上文案例中，显然`m`还引用这它，所以不回被回收，这就是「强引用」（只要`{a: 1}`被引用，那么它就不会被回收）。

那么，`WeakMap`又是怎么回事呢？和`Map`有什么区别呢？

```js
let obj1 = {
  a: 1
};

let obj2 = obj1;

const wm = new WeakMap();
wm.set(obj1, 1);

obj1 = null;
obj2 = null;

// 10 秒后打印 wm
setTimeout(() => {
  console.log(wm);
}, 10000);
```

`wm`虽然和`m`一样都引用了`{a: 1}`，但`WeakMap`是弱引用，也就是说当`obj1`、`obj2`解除了对`{a: 1}`的引用后，`wm`也将自动解除。

`WeakMap`引用`{a: 1}`的时候不会被垃圾回收机制所标记，只要强引用没有了，那么弱引用也就自动被回收！！！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1687674496791-d4433bed-2ebc-48d0-987e-09fc3651eb68.png)

所以，使用`WeakMap`的时候，`{a: 1}`就是被引用了 2 次，而不再是 3 次。

另外，垃圾回收机制的执行时机是不可预测的，有的时候控制台可以看到`wm`的值，有的是看不到。所以这里才通过定时器的方式去打印`wm`的值。

因为`WeakMap`是弱引用，它不知道自己的键值什么时候就会被回收掉了，所以`WeakMap`没有遍历的方法！！！

## 🔢 和 Set、Map 的区别
`WeakMap`和`WeakSet`与`Map`和`Set`对象用法基本上一致，简单理解就是削弱版的`Map`和`Set`

```js
const wm = new WeakMap();
const ws = new WeakSet();
```

区别：

1、`WeakMap`和`WeakSet`的数据成员只能是对象。

```js
let ws = new WeakSet();
ws.add({ name: 1 }); // 正常
ws.add(1); // Invalid value used in weak set
```

```js
let wm = new WeakMap();
wm.set({ name: 1 }, "111");
wm.set(1, 1); // Invalid value used as weak map key
```

2、`WeakMap`和`WeakSet`是使用弱引用的集合类型，存储在其中的对象不会阻止垃圾回收。当对象只在`WeakMap`或`WeakSet`中被引用且没有其他引用时，垃圾回收机制会忽略这些弱引用，将该对象回收，并自动从`WeakMap`或`WeakSet`中移除。

3、因为`WeakMap`和`WeakSet`内部的对象不适合被引用，也不允许遍历，因为有可能遍历的过程，内部的对象被垃圾回收机制回收了，所以没有遍历的方法。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1662361952233-f45ddc3f-8c5d-4b6c-83e9-58d7e889e45f.png)

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/02%20ECMAScript6+/_assets/1662361969826-1242c0cb-fdbe-484e-9b07-b43c6a9ebd1f.png)
