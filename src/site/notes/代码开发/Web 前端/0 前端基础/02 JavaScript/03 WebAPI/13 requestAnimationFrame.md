---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/13-request-animation-frame/","dg-note-properties":{}}
---

早期在 JS 中制造动画通常需要使用`setInterval()`来控制动画执行。

```js
function updateAnimations() {
  doAnimation1();
  doAnimation2();
  // 其他任务
}
setInterval(updateAnimations, 100);
```

这种定义动画的方式的问题在于无法准确知晓循环之间的延时。定时间隔必须足够短，这样才能让动画执行起来更加平滑。但是又要足够长，以便让浏览器可以渲染出来变化。

一般的显示器刷新率为 60HZ，也就是每 1s 都要重新绘制 60 次。因此也就得到了`1000/60 = 17`，约等于 17ms。使用这个间隔可以实现最平滑的动画，这已经是浏览器的极限了。

使用`setInterval()`虽然可以实现动画了，但是并不能保证「时间精度」。`setInterval()`的第二个参数只能保证什么时候把回调函数添加到浏览器的「任务队列」中去，而不能保证添加到队列后会立即运行。如果队列中有其他任务，那么这个定时任务就得需要进行排队。

## 🔢 时间间隔问题

知道什么时候绘制下一帧是创建平滑动画的关键。

随着 HTML5 和 Canvas 的流行，开发者发现`setTimeout()`和`setInterval()`时间不精确是个大问题。

- IE8 及更早版本的计时器精度为 15.625 毫秒； 
- IE9 及更晚版本的计时器精度为 4 毫秒； 
- Firefox 和 Safari 的计时器精度为约 10 毫秒； 
- Chrome 的计时器精度为 4 毫秒。

IE9 之前版本的精度是 15.625 毫秒，这就意味着 0~15 范围内的任何值最终要么是 0，要么是 15，不可能是别的数。IE9 把计时器的进度改为为 4 毫秒，但这对于动画而言还是不够精准。更麻烦的是，浏览器对于切换到后台或者不活跃的 Tab 标签页中的计时器执行限流，所以就算把间隔时间设置为最优，也避免不了类似的结果。

## 🔢 requestAnimationFrame
`requestAnimationFrame()`方法源自于 Mozilla。其核心原理是浏览器知道 CSS 的过渡动画应该什么时候开始，并计算出正确的时间间隔。那么对于 JS 只需要让浏览器在开始执行动画的时候“通知” JS 就可以了，这样浏览器就可以再运行某些代码的时候进行适当的优化。

`requestAnimationFrame()`方法接收一个参数，这个参数是一个要在屏幕重绘前调用的函数（一般是绘制动画的函数）。

为了实现动画循环，可以把`requestAnimationFrame()`的调用串联起来，就像是使用`setTimeout()`一样，因为该方法只会调用一次传入的动画函数，同时也需要控制动画的暂停。

```js
function updateProgress(time) {
  var div = document.getElementById("status");
  div.style.left = parseInt(div.style.left, 10) + 5 + "%";
  // 持续调用
  if (div.style.left != "100%") {
    requestAnimationFrame(updateProgress);
  }
}
requestAnimationFrame(updateProgress);
```

传递给`requestAnimationFrame()`的动画动画函数，实际上可以接收一个参数，该参数表示下次重绘的时间。

```js
function updateProgress(time) {
  console.log("🚀 ~ updateProgress ~ time:", time)

  var div = document.getElementById("status");
  div.style.left = parseInt(div.style.left, 10) + 5 + "%";
  // 持续调用
  if (div.style.left != "100%") {
    requestAnimationFrame(updateProgress);
  }
}
requestAnimationFrame(updateProgress);
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1773746545555-080e8aab-316e-4d3a-aa7d-e1933d924770.png)

## 🔢 cancelAnimationFrame
和定时器函数一样，`requestAnimationFrame()`方法也会返回一个请求 ID，调用`cancelAnimationFrame()`并传入 ID 可以取消重绘任务。

```js
let requestID = window.requestAnimationFrame(() => {
  console.log('Repaint!');
});
window.cancelAnimationFrame(requestID);
```

## 🔢 通过 requestAnimationFrame 节流
requestAnimationFrame 不只是一个动画执行的 API，在浏览器的内部，其实有一个「回调函数列表」，浏览器在每一帧准备绘制页面之前，都会把这个列表中的函数全部执行一遍。

每次调用`requestAnimationFrame(fn)`的时候其实就是往这个列表中添加一个`fn`函数，然后等待浏览器要开始重新绘制的时候，再把列表中的`fn`都清空。

当我们在`requestAnimationFrame(fn)`中递归调用`requestAnimationFrame()`的时候，可以保证每次重绘最多调用一次回调函数，这起到了非常好的节流作用。在频繁更改页面外观的时候，这个利用这个回调队列进行节流。

❌ Bad case：

```js
window.addEventListener('scroll', () => {
  console.log('执行了');
});

/** 滚动的时候 */
// 执行了
// 执行了
// 执行了
// 执行了
// 执行了
// ...（非常多）
```

✅ Good case：

```js
let ticking = false;

window.addEventListener('scroll', () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      console.log('真正执行');
      ticking = false;
    });
    ticking = true;
  }
});
```

使用`requestAnimationFrame()`实现节流，只有当重新绘制完成后才会继续执行`if`块中的代码。
