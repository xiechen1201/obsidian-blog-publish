---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/4 前端框架/React18/26 diff 算法/","dg-note-properties":{}}
---

在 React 渲染流程中，Render （更准确地说是 reconcile 阶段 ）阶段会生成 Fiber Tree，所谓的 diff 实际上就是发生在这个阶段，==这里的 diff 指的是 current FiberNode 和 React Element 对象之间进行对比，然后生成新的 workInProgress FiberNode。==

> 除了 React 之外，其他使用虚拟 DOM 的前端框架也会有类似的流程，例如 Vue 里面的这个流程称为 Patch。

diff 算法本身是有性能消耗的，在 React 文档中也有提到，即使采用最前沿的算法，如果要完整地对比两棵树，那么算法的复杂度都会达到 O(n^3)，n 代表的是元素的数量，如果 n 为 100，那么就要执行的计算量会达到十亿量级。

所以，为了降低算法的复杂度，React 为 diff 算法设置了 3 个限制：
- 限制一：只对同级别的元素进行 diff，<u>如果一个 DOM 元素在前后两次更新中跨越了层级，那么 React 不会尝试进行复用；</u>
- 限制二：两个不同类型的元素会产生不同的树。<u>例如元素从 div 变成了 p，那么 React 就会直接销毁 div 以及子孙元素，新建 p 以及 p 的子孙元素；</u>

更新前：

```jsx
<div>
  <p key="one">one</p>
  <h3 key="two">two</h3>
</div>
```

更新后：

```jsx
<div>
  <h3 key="two">two</h3>
  <p key="one">one</p>
</div>
```

如果没有 key，那么 React 就会认为 div 的第一个子元素从 p 变成了 h3，第二个子元素从 h3 变成了 p，因此 React 就会采用上面提到的限制二的规则。

但是如果使用了 key，那么此时的 DOM 元素就可以进行复用，只不过是前后的位置进行了互换。

回头再看限制一，对同级元素进行 diff 究竟是如何进行 diff 的？整个 diff 流程可以分为两大类：
- 更新后只有一个元素，此时就会根据 newChild 创建对应的 workInProgress FiberNode，对应的流程是==单节点 diff==；
- 更新后有多个元素，此时就会遍历 newChild 创建对应的 workInProgress FiberNode 以及它的兄弟元素，此时对应的流程就是==多节点 diff==；

## 单节点 diff

单节点 diff 就是说新的节点只有一个节点，但是旧的节点数量是不一定的。

单节点 diff 流程如下：
- 判断 key 是否相同
	- 如果更新前后没有设置 key，则 key 为 null，此时也属于 key 相同；
	- 如果 key 相同，进入步骤二；
	- 如果 key 不同，不需要往下走流程，不需要判断 type 是否一致，结果直接为不可复用（如果有兄弟节点还会继续遍历兄弟节点）；
- 如果 key 相同，再判断 type 是否一致；
	- 如果 type 一致，则可以复用；
	- 如果 type 不同，无法进行复用（并且兄弟节点也会一并被标记为删除）；

示例一，更新前：

```jsx
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>
```

更新后：

```jsx
<ul>
  <p>1</p>
</ul>
```

上面的示例中，因为没有设置 key 属性，所以会被视为 key 是相同的，接下来就会进入到 type 的判断。此时发现 type 不同，因此不能进行复用。

示例中，唯一的可能性（指的是 li 元素）都不可以进行复用，那么就会标记兄弟 FiberNode 为删除状态。

示例二，更新前：

```jsx
<div>one</div>
```

更新后：

```jsx
<p>one</p>
```

上面的示例中，前后两次依旧没有设置 key 属性，因此依旧被认为 key 是一样的。接下来就是对比 type，发现 type 也不相同，因此不能进行复用。

示例三，更新前：

```jsx
<div key="one">one</div>
```

更新后：

```jsx
<div key="two">one</div>
```

在这个示例中，虽然都加上了 key 属性，但是前后两次 key 不同，结果依旧是不能进行复用。

示例四，更新前：

```jsx
<div key="one">one</div>
```

更新后：

```jsx
<div key="one">two</div>
```

首次先判断 key 相同，再次判断 type 也相同，这个 FiberNode 是可以进行复用的，children 是一个文本节点，之后只需要更新文本节点即可。

## 多节点 diff

所谓多节点 diff 指的是新节点有多个。

React 团队发现，在日常开发中，对节点的「更新操作」往往要大于对节点的「新增、删除、移动」，因此在进行多节点 diff 的时候，React 会进行两轮遍历：
- 第一轮：尝试逐个的复用节点；
- 第二轮：处理第一轮遍历中没有处理完的节点；

### 第一轮遍历

第一轮遍历从前往后依次进行遍历，存在三种情况：
- 如果新旧子节点的 key 和 type 都相同，说明可以进行复用；
- 如果新旧子节点的 key 相同，但是 type 不相同，这个时候就会根据 React Element 来生成一个新的 FiberNode，旧的 FiberNode 会被放入到 deletions 数组中，后面统一进行删除。但是需要注意，此时遍历并不会终止；
- 如果新旧子节点的 key 和 type 都不相同，结束遍历；

示例一，更新前：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

更新后：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="e">e</div>
  <div key="d">d</div>
</div>
```

首先会遍历到 div.key.a，发现这个 FiberNode 能够进行复用：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-032654.png)
继续往后走，发现 div.key.b 也可以进行复用：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-075146.png)
接下来继续往后走，发现 div.key.e 这个 key 和旧 FiberNode 不相同，因此第一轮遍历就结束了。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-075345.png)
示例二，更新前：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

更新后：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <p key="c">c</p>
  <div key="d">d</div>
</div>
```

首先依旧是先对比 div.key.a 和 div.key.b 这两个 FiberNode 可以进行复用。接下来对比第三个 FiberNode 的时候，发现 key 是相同的，但是 type 不同，此时就会将对应的旧 FiberNode 放入一个叫做 deletions 的数组中，后面统一进行删除，然后根据新的 React Element 创建一个新的 FiberNode，此时的遍历没有结束。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-075011.png)
接下来继续往后面进行遍历，遍历什么时候结束呢？
- 到末尾了，也就是说整个遍历完成了；
- 或者和示例一一样，新旧 FiberNode 的 key 不相同；
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-075725.png)
### 第二轮遍历

如果第一轮遍历被提前终止，那么就意味着有新的 React Element 或者旧的 FiberNode 没有遍历完，此时就会进行第二轮遍历。

第二轮遍历会处理这么三种情况：
- 只剩下旧子节点：将旧子节点添加到 deletions 数组里面直接删除掉（删除的情况）；
- 只剩下新的 Element 元素：根据 React Element 元素来创建 FiberNode 节点（新增的情况）；
- 新旧子节点都有剩余：会将剩余的旧 FiberNode 节点放入一个 map 里面，然后遍历剩余的新的 React Element 元素，从 map 里面寻找能够复用的 FiberNode 节点。如果能找到就拿来复用（移动的情况），如果找不到那就新增。当新的 React Element 元素都遍历完成了，此时如果 map 中还有旧的 FiberNode，那么就把这些 FiberNode 添加到 deletions 数组里面，之后统一做删除操作；

1、只剩下旧子节点

更新前：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

更新后：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
</div>
```

遍历前两个节点的时候发现可以进行复用，此时就会复用前面的节点，对于 React Element 来说，遍历完前面两个节点就已经遍历完成了，因此剩余的 FiberNode 就会被放入到 deletions 数组里面，后面统一删除。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-080358.png)
2、只剩下新的 React Element 元素

更新前：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
</div>
```

更新后：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

根据新的 React Element 创建对应的 FiberNode 即可。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-080558.png)
3、新旧子节点都有剩余

更新前：

```jsx
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

更新后：

```jsx
<div>
  <div key="a">a</div>
  <div key="c">c</div>
  <div key="b">b</div>
  <div key="e">e</div>
</div>
```

首先会把剩余的旧 FiberNode 放入一个 map 里：
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-081414.png)
接下来会继续遍历剩余的 React Element 对象数组，遍历的同时从 map 里面去找有没有可以复用的 FiberNode。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-081859.png)
如果 map 里面没有找到，那么就会创建这个 FiberNode，如果整个 React Element 对象数组遍历完成后，map 里面仍然存在剩余的 FiberNode，说明这些 FiberNode 是无法进行复用的，那么直接放入到 deletions 数组里面，后期统一进行删除。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-082152.png)
## 和双端算法对比

所谓双端 diff 指的就是在新旧子节点数组中，各有两个指针指向头尾的节点，在遍历的过程中，头尾两个指针同时向中间靠拢。

因此在新节点数组中，会有两个指针，newStartIndex 和 newEndIndex 分别指向新子节点数组的头和尾。
在旧子节点数组中，也会有两个指针，oldStartIndex 和 oldEndIndex 分别指向旧子节点数组的头和尾。
![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/4%20%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6/React18/_assets/2023-02-28-085007.png)
每遍历到一个节点，就会尝试进行双端对比：「新头 VS 旧头」、「新尾 VS 旧尾」、「新头 VS 旧尾」、「新尾 VS 旧头」。
如果匹配成功，更新双端的指针。例如，新旧子节点通过「新头 VS 旧尾」匹配成功，那么 `newStartIndex + 1`、`oldEndIndex - 1`。
如果新旧子节点通过「新尾 VS 旧头」匹配成功，还需要将「旧头」对应的 DOM 节点插入到「旧尾」对应的 DOM 之前。
如果新旧子节点通过「新头 VS 旧尾」匹配成功，还需要将「旧尾」对应的 DOM 节点插入到「旧头」对应的 DOM 之前。

在 React 的源码中，还解释了为什么没有使用双端 diff 算法：

```js
function reconcileChildrenArray(
returnFiber: Fiber,
 currentFirstChild: Fiber | null,
 newChildren: Array<*>,
 expirationTime: ExpirationTime,
): Fiber | null {
    // This algorithm can't optimize by searching from both ends since we
    // don't have backpointers on fibers. I'm trying to see how far we can get
    // with that model. If it ends up not being worth the tradeoffs, we can
    // add it later.

    // Even with a two ended optimization, we'd want to optimize for the case
    // where there are few changes and brute force the comparison instead of
    // going for the Map. It'd like to explore hitting that path first in
    // forward-only mode and only go for the Map once we notice that we need
    // lots of look ahead. This doesn't handle reversal as well as two ended
    // search but that's unusual. Besides, for the two ended optimization to
    // work on Iterables, we'd need to copy the whole set.

    // In this first iteration, we'll just live with hitting the bad case
    // (adding everything to a Map) in for every insert/move.

    // If you change this code, also update reconcileChildrenIterator() which
    // uses the same algorithm.
}
```

上面的注释就是说：由于双端 diff 需要向前查找节点，但每个 FiberNode 节点上都没有反向指针，即前一个 FiberNode 通过 sibling 属性指向后一个 FiberNode，只能从前往后遍历，而不能反过来，因此该算法无法通过双端搜索来进行优化。

因为 Fiber 是使用链表的形式：

```js
fiber.child
fiber.sibling
fiber.return
```

没有 previousSibling 属性，因此只能从前往后找，而不能反过来。

React 想看下现在用这种方式能走多远，如果这种方式不理想，以后再考虑实现双端 diff。React 认为对于列表反转和需要进行双端搜索的场景是少见的，所以在这一版的实现中，先不对 bad case 做额外的优化。
