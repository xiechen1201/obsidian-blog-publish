---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/14-form-data/","dg-note-properties":{}}
---

提到文件上传就不得不表单的`input`，`input`标签的属性`type="file"`的时候表示标签这是一个文件上传的表单项

```html
<form action="./index.php" method="post">
  <input type="file" name="file" />
  <input type="submit" value="提交" />
</form>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1660283583981-c854b4c1-42dc-443e-b9c4-b4b18810b133.png)

当我们点击「选择文件」的时候浏览器会唤起选择文件的操作框，点击「提交」的时候会把表单数据直接发送到后端的接口（示例中`./index.php`）

例如我选择一个文件后进行提交：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1660283815299-3f7d4f4a-167c-4d25-89d1-a1b461f1db08.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1660283775400-36326906-4f28-4a73-8585-6d56854fc624.png)

可以看到请求头里的数据格式是`Content-Type: application/x-www-form-urlencoded`，这表示是以「表单」的方式进行提交的，对表单数据进行序列号，这也是`post`请求默认的方式！！！（我们常用的`axios`工具库默认提交都是`JSON`的方式）

这个时候我们能发现，哎？不对啊，我刚才选择的不是一个文件吗？怎么`Form Data`里面只有一个文件的名字呢？？？

其实`application/x-www-form-urlencoded`请求数据的时候真正的样子是`param1=xxx&param2=xxx`，浏览器只是帮我们进行了美化，当我们点击`view source`的时候就能看到原始数据的格式

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1660284126538-e46077e7-8b76-4604-94a6-e193eafd8893.png)

`application/x-www-form-urlencoded`格式请求接口的时候只能传输文本的数据，当我们上次一个文件的时候，请求找不到文本数据就只能找相应的标识，也就是文件名。

那么如何才能上传文件呢？

可以使用「二进制」的方式将文件分割为字符串进行传递。

`form`标签还有个属性`enctype`用来设置表单提交时的数据格式，我们只需要将`enctype`设置为`multipart/form-data`就可以传输文件了。

`multipart/form-data`和`application/x-www-form-urlencoded`都是表单的格式进行提交，只不过`multipart/form-data`可以传递文件，而`enctype`属性默认就是`application/x-www-form-urlencoded`

```html
<form action="./index.php" method="post" enctype="multipart/form-data">
  <input type="file" name="file" />
  <input type="submit" value="提交" />
</form>
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1660284490282-6897c777-9a60-4a91-a2b4-43b5a3976f11.png)

> 我这里没有后端服务只能将就着看了🥲
>

如何上传多个文件呢？

`name`的值设置为`file[]`就表示是一个数组，用来传递多个文件。

```html
<form action="./index.php" method="post" enctype="multipart/form-data">
    <input type="file" name="file[]" multiple />
    <input type="submit" value="提交" />
  </form>
```

## 🔢 FromData()
以上都是基于`form`进行的表单上传文件然后同步提交数据，而现在我们开发的时候基本上都是异步请求，那么如何使用`Ajax`进行上传文件呢？

再说`Ajax`上传文件之前，我们必须要认识一个构造函数`FormData()`。

`FormData()`是表单`form`的表现方式，和`Image()`构造函数一样可以创建一个图片标签，`FormData()`用于创建一个表单标签。

```js
var form = new FormData();
```

操作`FormData`必须使用实例方法：

` append("name","value")`：往表单里添加表单项

` get("name")`：获取表单数据

` set("name","value")`：设置表单数据

` has("name")` ：查询是否存在某个表单项，返回布尔值

` delete("name")`：删除表单项

<br/>

```js
var formData = new FormData();
formData.append("user", "张三");
console.log(formData.get("user")); // 张三
formData.set("age", 20);
console.log(formData.has("age")); // true
formData.delete("age");
console.log(formData.has("age")); // false
```

需要特别注意的是直接打印`formData`是看不到任何数据的，必须使用`get()`方法才能看到数据：

```js
var formData = new FormData();
formData.append("user", "张三");
console.log(formData);
console.log(formData.get('user'));
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1660285215563-0d819696-6f48-49b6-80ce-e359b96a299a.png)

## 🔢 Ajax 上传文件
接着上面的文件上传，我们需要有一个`input`来选择文件

```html
<input type="file" name="file" id="file" />
```

```js
var oFlie = document.getElementById("file");

oFlie.onchange = function (e){
  // this.files 所选文件的伪数组，每项包含文件的相关的信息，fileSize 是字节单位
  console.log(this.files)
}
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1660285742349-5e3a5731-b668-42d6-a248-d53e8c1216da.png)

然后我们就要实例化`FormData()`进行添加数据：

```js
var oFlie = document.getElementById("file");

oFlie.onchange = function (e){
  var formData = new FormData();
  // file 是一个字段名，根据实际业务更改！！！
  formData.append("file", this.files[0]);
}
```

最后我们调用`Ajax`发送请求：

```js
var oFlie = document.getElementById("file");

oFlie.onchange = function (e){
  var formData = new FormData();
  // file 是一个字段名，根据实际业务更改！！！
  formData.append("file", this.files[0]);

  requestAjax(formData);
}

function requestAjax(formData) {
  // 实例化 XMLHttpRequest()
  var xhr = new XMLHttpRequest();
  xhr.open("post", "./index.php");
  // 设置请求头
  xhr.setRequestHeader("Content-type", "multipart/form-data");
  xhr.send(formData);
  xhr.onreadystatechange = function () {
    if (xhr.readyState == 4 && xhr.status == 200) {
      alert("上传成功");
    } else {
      alert("上传失败");
    }
  };
}

```

这样就实现了一个简单的文件上传功能。

如果想要知道上传进度，我们还可以使用`Ajax`的进度事件`onprogress`：

```js
function requestAjax(formData) {
    var xhr = new XMLHttpRequest();
    xhr.open("post", "./index.php");
    // 设置请求头
    xhr.setRequestHeader("Content-type", "multipart/form-data");
    xhr.send(formData);
    // 在接收响应期间持续不断地触
    xhr.onprogress = function (e) {
      console.log(e.loaded); // 返回已经上传的字节数
      console.log(e.total); // 返回总的字节数
      var percent = (e.loaded / e.total) * 100 + "%";
      console.log("已上传：" + percent);
    };
    // onload 会在请求完成后触发
    xhr.onload = function () {
      // 对返回的 xhr.responseText 进行一些判断处理
      // ...
    };
  }
```
