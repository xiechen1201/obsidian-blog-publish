---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/2 类型系统/TypeScript/05 类和接口/06 this 作为返回类型/","dg-note-properties":{}}
---

---
---
`this`在类中可以作为值也可以作为方法的返回类型。

例如实现一个简化版的`Set`：

```ts
class SimpleSet {
    private elements: Map<number, boolean>;
    constructor() {
        this.elements = new Map();
    }

    // 返回 SimpleSet 类型
    add(element: number): SimpleSet {
        this.elements.set(element, true);
        return this;
    }

    has(element: number): boolean {
        return this.elements.has(element);
    }

    delete(element: number): boolean {
        return this.elements.delete(element);
    }

    values(): number[] {
        return Array.from(this.elements.keys());
    }
}
```

然后我们进行实例化这个类：

```ts
let mySet = new SimpleSet();
mySet.add(1).add(2).add(3); // 链式调用
console.log(mySet.values()); // [1, 2, 3]
```

以上代码运行是没有问题的，但是如果我们还需要实现一个子类，子类可能还需要处理其他的内容。

例如：

```ts
class MutableSet extends SimpleSet {
    show() {
        console.log("MutableSet Class");
    }
}
```

接着我们进行实例化`MutableSet`类，并调用`SimpleSet`类的原型方法`add`：

```ts
const mySet = new MutableSet();

mySet.add(1).add(2).add(3).show(); // ❌ 类型“SimpleSet”上不存在属性“show”
//
```

以上代码中`add`方法返回的是`SimpleSet`类, `show`是`MutableSet`的方法，所以根本调用不到`show`方法。

所以，为了保证子类的`this`返回正确，我们可以重写父类的`add`方法，目的仅仅是改写返回的对象类型：

```ts
class MutableSet extends SimpleSet {
    add(element: number): MutableSet {
        super.add(element);
        return this;
    }

    show() {
        console.log('MutableSet show')
    }
}
```

其实还有一种更加简单的办法就能解决，那就是直接将`SimpleSet`类的`add`方法类型更改为`this`，这样就可以指向`MutableSet`了：

```ts
class SimpleSet {
    ...
    add(element: number): this {
        this.elements.set(element, true);
        return this;
    }
}
```

```ts
myMutableSet.add(1).add(2).add(3).show(); // ✅
```

这样就可以安全的调用了。
