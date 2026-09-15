---
{"dg-publish":true,"permalink":"//web/0/02-java-script/01-java-script/08-object/01-object-define-property-define-properties/","dg-note-properties":{}}
---

`Object.defineProperty()`和`Object.defineProperties()`都是`Object`构造函数的静态方法，作用是定义数据！！！

## 🔢 defineProperty()

参数1: 需要被定义的对象

参数2: 需要被定义的对象属性

参数3: 描述配置对象

<br/>

```js
// 假如我现在要对 obj 对象的 test 属性进行定义

var obj = {};
Object.defineProperty(obj, "test", {
  value: undefined // value 属性表示 test 属性的值，后面还会讲到其他的配置属性
});
console.log(obj); // {test: undefined}
```

以上代码的`obj`从明面上看和字面量定义的对象没有任何的区别：

```js
var obj = {
  test: undefined
}
console.log(obj); // {test: undefined}
```

## 🔢 属性配置
可事实真的如此吗？下面我们将两种方式对象进行对比。

```js
var obj1 = {a: 1};
var obj2 = Object.defineProperty({}, "b", {
  value: 2
});

console.log(obj1); // {a: 1}
console.log(obj2); // {b: 2}
```

这样我们就生成了`obj1`和`obj2`两个对象。

我们想两个对象的属性值进行修改尝试：

```js
obj1.a = 2;
obj2.b = 3;

console.log(obj1); // {a: 2}
console.log(obj2); // {b: 2}
```

<br/>color5
结果：使用`Object.defineProperty()`定义的属性无法更改属性值。

<br/>

然后我们去遍历两个对象：

```js
for (const key in obj1) {
  console.log(obj1[key]);
}
// 1

for (const key in obj2) {
  console.log(obj2[key]);
}
// 无任何输出
```

<br/>color5
结果：使用`Object.defineProperty()`定义的属性无法枚举

枚举就是将项列举出来。

<br/>

最后我们去删除两个对象的属性：

```js
delete obj1.a;
delete obj2.b;

console.log(obj1); // {}
console.log(obj2); // {b: 2}
```

<br/>color5
结果：使用`Object.defineProperty()`定义的属性无法删除

<br/>

🌴 总结：

使用`Object.defineProperty()`定义的属性无法更改、删除和遍历。

<br/>

到现在我们发现`Object.defineProperty()`的参数 3 一直没有配置，其实参数 3 是个对象用于对对象的属性进行相关的配置：

```js
var obj = Object.defineProperty({}, "test",{
  value: 1, // 配置属性的值
  writable: true, // 配置属性是否可以被更改，默认是 false
  enumerable: true, // 配置属性是否可以被枚举，默认是 false
  configurable: true // 配置属性是否可以被删除，默认是 false
})

// 然后在对 obj 的属性进行操作
console.log(obj); // {test: 1}

// 更改
obj.test = 123;
console.log(obj); // {test: 123}

// 遍历
for (const key in obj) {
  console.log(obj[key]);
}
// 123

// 删除
delete obj.test;
console.log(obj); // {}
```

## 🔢 get()/set()
每个属性在定义的时候都会存在`getter`和`setter`的机制，所以`Object.defineProperty()`的参数 3 还有`set()`和`get()`方法。

```js
var value = 1;
var obj = Object.defineProperty({}, "test",{
  get: function(){
    console.log("访问 obj.test 属性")
    return value;
  },
  set: function(newVal){
    console.log("设置 obj.test 属性")
    value = newVal;
  }
})

console.log(obj.test);
// 访问 obj.test 属性
// 1

obj.test = 456;
// 设置 obj.test 属性

console.log(obj.test);
// 访问 obj.test 属性
// 456

consoloe.log(obj);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1657716646755-7e66969d-44a1-4873-a3ac-ce896b7b134e.png)

这里可以发现`test`属性是展开三角后才能看到的，猜测这是因为浏览器要和普通属性作区分特意而为之。

利用`get`和`set`方法就可以实现对对象属性的数据劫持！！！

```js
var obj = {};
Object.defineProperty(obj, "a", {
  get() {
    return "this is a";
  },
  set(newVal) {
    // 当我们对象 obj.a 进行设置时候，p 元素的内容也会发生变化
    var oP = document.getElementsByTagName("p")[0];
    oP.innerHTML = newVal;
  }
});
```

`defineProperty()`本身是`Object`构造函数的静态方法，所以该方法本身只可以对对象具备劫持，那么如何对数组也进行劫持呢？

```js
function DateArr() {
  var _arr = [],
      _val = null;

  Object.defineProperty(this, "val", {
    get: function () {
      return _val;
    },
    set: function (newVal) {
      _val = newVal;
      _arr.push({ val: _val });
      console.log("A new value " + _val + "has been pushed to _arr");
    },
  });

  this.getArr = function () {
    return _arr;
  };
}

var dateArr = new DateArr();
dateArr.val = 123;
dateArr.val = 234;
console.log(dateArr.getArr()); // [{val: 123}, {val: 234}]
```

## 🔢 互斥
<br/>danger
注意 ⚠️

如果配置中只有`enumerable`、`configurable`的时候该配置是对数据描述。

如果配置中同时存在 「`value`、`writable`」和 「`get`、`set`」 是互斥的！！！

<br/>

```js
var obj = {};

// 以下的情况会发生错误
var obj1 = Object.defineProperty({}, "a" ,{
  value: 1, // error
  writable: true, // error
  get() {
    return "this is a";
  },
  set(newVal) {
    var oP = document.getElementsByTagName("p")[0];
    oP.innerHTML = newVal;
  }
});

// 以下的情况会正常执行
var obj2 = Object.defineProperty({}, "a" ,{
  value: 1,
  writable: true,
  enumerable: true,
  configurable: true,
});

// 以下的情况也会正常执行
var obj3 = Object.defineProperty({}, "a" ,{
  enumerable: true,
  configurable: true,
  get() {
    return "this is a";
  },
  set(newVal) {
    var oP = document.getElementsByTagName("p")[0];
    oP.innerHTML = newVal;
  }
});
```

## 🔢 defineProperties()
`Object.defineProperty()`方法只能对对象的一个属性进行定义，那么如何定义对象的多个属性呢？

可以使用`Object.defineProperties()`来定义多个属性。

`defineProperties()`和`defineProperty()`写法的不同在于`defineProperties`接收两个参数，参数1 是要定义的对象，参数 2 是定义对象属性的对象。

```js
var value = 2;

var obj = Object.defineProperties({}, {
  a: {
    value: 1,
    writable: true,
    enumerable: true,
    configurable: true,
  },
  b: {
    get() {
      return value;
    },
    set(newVal) {
      value = newVal;
    },
  }
});

console.log(obj);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/01%20JavaScript%20%E5%9F%BA%E7%A1%80/08%20Object%20%E5%AF%B9%E8%B1%A1/_assets/1657717869532-ba9254dc-9644-4377-987f-0201a990c56d.png)
