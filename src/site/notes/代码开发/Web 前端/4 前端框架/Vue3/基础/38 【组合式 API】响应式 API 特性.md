---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/Vue3/基础/38 【组合式 API】响应式 API 特性/","dg-note-properties":{}}
---

## reactive()

在组合式 API 中，如果我们想要创建一个响应式的对象（数据发生变化视图也会变化就是响应式）需要使用`reactive()`方法。

```js
import { reactive } from "vue";

export default {
  setup() {
    const data = {
      a: 1,
      b: {
        c: 2
      }
    };
    const state = reactive(data);
    console.log(state);
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686552414945-68cc47ee-4f19-4ab1-b0c6-bc3fee733d9b.png)

`reactive()`方法返回一个「代理对象」，该代理对象和「原对象」不是同一个引用。只有代理对象是响应式的，更改原始对象不会触发更新。

```js
setup() {
  const data = {
    a: 1,
    b: {
      c: 2
    }
  };
  const state = reactive(data);
  console.log(state == data); // false
}
```

当一个被`reactive()`方法「包装过」的对象再次被包装，会返回第一次被包装的代理商对象。简单说就是无论包装多少次都会返回第一次被包装的`state`对象，是相同的引用：

```js
setup() {
  const data = {
    a: 1,
    b: {
      c: 2
    }
  };
  const state = reactive(data);
  const state1 = reactive(state);
  const state2 = reactive(state);

  console.log(state === state1); // true
  console.log(state === state2); // true
}
```

## isReactive()

可以通过`isReactive()`方法来判断一个对象是不是`reactive()`包装过的对象：

```js
import { reactive, isReactive } from "vue";

export default {
  setup() {
    const data = {
      a: 1,
      b: {
        c: 2
      }
    };
    const state = reactive(data);
    console.log(isReactive(state)); // true
  }
};
```

## shallowReactive()

`shallowReactive()`和`reactive()`方法作用相同也是把一个对象包装为一个响应式的数据。但`shallowReactive()`包装后的数据是浅层响应式的，`reactive()`是深层的。

```js
import { reactive, shallowReactive, isReactive } from "vue";

export default {
  setup() {
    const data = {
      a: 1,
      b: {
        c: 2
      }
    };

    const state = shallowReactive(data);

    // 使用 isReactive() 来判断
    console.log(isReactive(state)); // true
    console.log(isReactive(state.b)); // false

    setTimeout(() => {
      state.b.c = "Test";

      console.log(state)
    }, 1000);

    return {
      state
    };
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686555570913-928d81db-6b1a-4588-a767-bcbfceb1f62c.gif)

可以看到，1 秒后数据发生了变化，但是页面没有进行更新！！！

## reactive() 的局限性

1、只能针对 Array、Map、Set、Object 这样的引用类型有效，无法包装 String、Number、Boolean 这样的原始类型。

2、因为 Vue 的响应式系统是通过属性访问进行追踪的，因此我们必须始终保持对该响应式对象的相同引用。这意味着我们不可以随意地“替换”一个响应式对象，因为这将导致对初始引用的响应性连接丢失：

```js
let state = reactive({ count: 0 })

// 上面的引用 ({ count: 0 }) 将不再被追踪（响应性连接已丢失！）
state = reactive({ count: 1 })
```

所以通常需要包一个最外层大对象：

```js
let state = reactive({
  data: { count: 0 }
})
state.data = { count: 1 }; // 这样 data 始终都是响应式的
```

3、不能把属性进行解构单独使用，这样也会丢失响应式

```js
const state = reactive({ count: 0 })

// n 是一个局部变量，同 state.count
// 失去响应性连接
let n = state.count
// 不影响原始的 state
n++

// count 也和 state.count 失去了响应性连接
let { count } = state
// 不会影响原始的 state
count++

// 该函数接收一个普通数字，并且
// 将无法跟踪 state.count 的变化
callSomeFunction(state.count)
```

因为当我们把对象属性解构后使用，就单纯的拿到属性的值，不会再触发对象属性的 setter/getter 机制了。

所以，`ref()`函数就诞生了，`ref()`就是为了解决所有值类型都能使用 “引用” 机制。

## ref()

`ref()`是针对所有值的定制化的包装引用！包装响应式的同时还可以进行相应的操作和传递，例如解构、函数传参数。

```js
import { ref } from "vue";

export default {
  setup() {
    const title = ref("This is title.");
    console.log(title);
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686556973834-d14d9b66-930c-44b8-8d1b-16b27921526a.png)

通过`ref()`函数包装过的数据会返回一个对象，对象的属性分别有：

- `dep`：数据的依赖；
- `__v_isRef`：是否是 Ref 对象；
- `__v_isShallow`：是否是浅响应式；
- `_rawValue`：数据定义时的值；
- `_value`：当前动态的值；
- `value`：值

和响应式对象的属性类似，ref 的`.value`属性也是响应式的。同时，当值为对象类型时，会用`reactive()`自动转换它的`.value`。

```js
const objectRef = ref({ count: 0 });
console.log(objectRef);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686557218856-ba6a9dfc-a0cd-4269-862a-bbf2936c6bc0.png)

可以看到，`.value`属性依然一个被代理过的对象。

一个包含对象类型值的 ref 可以响应式地替换整个对象：

```js
const objectRef = ref({ count: 0 })

// 这是响应式的替换
objectRef.value = { count: 1 }
```

简言之，`ref()`让我们能创造一种对任意值的 “引用”，并能够在不丢失响应性的前提下传递这些引用。

## ref() 自动解包

## 在模版中解包

当 ref 在模板中作为顶层属性被访问时，它们会被自动“解包”，所以不需要使用`.value`。

```vue
<template>
  <button @click="increment">
    <!-- 无需 .value -->
    {{ count }}
  </button>
</template>

<script>
import { ref } from "vue";

export default {
  setup() {
    const count = ref(0);

    function increment() {
      count.value++;
    }

    return {
      count,
      increment
    };
  }
};
</script>
```

> [!warning]
>
> 请注意，仅当 ref 是模板渲染上下文的顶层属性时才适用自动“解包”。 例如， `object`是顶层属性，但`object.foo `不是。


```vue
<template>
  <button @click="increment">
    {{ object.foo + 1 }}
  </button>
</template>

<script>
  import { ref } from "vue";

  export default {
    setup() {
      const object = { foo: ref(1) };

      function increment() {
        object.count.value++;
      }

      return {
        object,
        increment
      };
    }
  };
</script>

```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686558343860-e313d3ec-3d64-4ddd-a730-aed18f02fb88.png)

这是因为`object.foo`是一个对象，她会调用`toString()`方法进行计算。

我们可以把`foo`进行解构后返回：

```vue
const { foo } = object
```

需要注意的是，如果一个 ref 是文本插值（即一个`{{ }}`符号）计算的最终值，它也将被解包。因此下面的渲染结果将为 1：

```js
{{ object.foo }}
```

## 在对象中解包

当使用`reactive()`包装一个「对象」的时，如果对象的属性值是一个 ref 对象，ref 对象会自动把`.value`提取出来，这和普通属性是一样的：

```js
import { ref, reactive } from "vue";

export default {
  setup() {
    const count = ref(0);
    const state = reactive({ count });

    console.log(count, state);
    // 不需要使用 state.count.value
    console.log(state.count);
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686557738500-c6297fe4-534e-4e90-9f3a-7302b03de6bc.png)

虽然进行了解包，但`count`依然是同步的:

```js
setup() {
  const count = ref(0);
  const state = reactive({ count });

  console.log(count, state.count); // ref， 0

  setTimeout(() => {
    count.value = 3;

    console.log(count, state.count); // ref， 3
  }, 1000);
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686557954034-9934d0e9-00cf-4697-81aa-7261ef605db8.gif)

跟响应式对象不同，当 ref 作为响应式数组或像 Map 这种原生集合类型的元素被访问时，不会进行自动解包。

```js
const books = reactive([ref('Vue 3 Guide')])
// 这里需要 .value
console.log(books[0].value);

const map = reactive(new Map([['count', ref(0)]]))
// 这里需要 .value
console.log(map.get('count').value)
```

## isRef()

我们可以使用`isRef()`方法来判断一个数据是不是 ref 对象：

```js
const count = ref(0);
console.log(count.value); // 0
console.log(isRef(count)); // true
```

## shallowRef()

和`ref()`方法一样，`shallowRef()`也可以把数据包装为 ref 对象，只不过是浅层的包装：

```js
import {  isRef, shallowRef } from "vue";

export default {
  setup() {
    const state = shallowRef({
      a: 1,
      b: {
        c: 2
      }
    });

    console.log(state);
    console.log(isRef(state)); // true
    console.log(isRef(state.b)); // false
  }
};
```

## toRef() 与 toRefs()

前面我们说过，当使用`reactive()`包装对象后我们不能把对象的属性进行解构使用，因为这会导致响应式的丢失，所以我们可以使用`toRefs()`把对象的属性全部转换为响应式的属性：

```js
import { ref, isRef, reactive, toRefs } from "vue";

export default {
  setup() {
    const state = reactive({
      a: 1,
      b: {
        c: 2
      }
    });
    const { a, b } = toRefs(state);
    console.log(a, b);
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686559023938-5b9a650c-247f-42b0-ad3a-ab5d667d7b78.png)

这样，`a`和`b`就都是响应式的啦。

如果你不想把所有的属性都进行解构，可以使用`toRef()`只把某一个属性进行包装为响应式的数据：

```js
import { ref, isRef, reactive, toRefs, toRef } from "vue";

export default {
  setup() {
    const state = reactive({
      a: 1,
      b: {
        c: 2
      }
    });
    const res = toRef(state, "a");
    console.log(res);
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686559206903-5d0fe0d3-65c5-4ab4-8a73-fbbbdac76cc9.png)

## unref()

如果参数是 ref，则返回内部值，否则返回参数本身。

这是`val = isRef(val) ? val.value : val`计算的一个语法糖。

```js
import { ref, unref } from "vue";

export default {
  setup() {
    const formData = ref({
      username: "xiechen",
      password: "123456"
    });

    console.log(formData, unref(formData));
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/Vue3/%E5%9F%BA%E7%A1%80/_assets/1686559835366-f461b88c-5681-425b-9dd0-aac2b9d1a3a0.png)
