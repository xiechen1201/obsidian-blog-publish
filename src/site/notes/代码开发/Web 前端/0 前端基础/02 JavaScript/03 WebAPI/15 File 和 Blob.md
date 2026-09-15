---
{"dg-publish":true,"permalink":"//web/0/02-java-script/03-web-api/15-file-blob/","dg-note-properties":{}}
---

## 🔢 File 类型
HTML5 在 DOM 上为文件输入元素添加了`files`集合。

当用户在文件字段中选择一个或多个文件时，这个`files`集合中会包含一组`File`对象，表示被选中的文件。

每个`File`对象都有一些只读属性：

- `name`：本地系统中的文件名。
- `size`：以字节计的文件大小。 
- `type`：包含文件`MIME`类型的字符串。 
- `lastModifiedDate`：表示文件最后修改时间的字符串。这个属性只有`Chome`实现了。

```js
let filesList = document.getElementById("fileInput");

filesList.addEventListener("change", (event) => {
  let files = event.target.files,
    	i = 0,
    	len = files.length;

  while (i < len) {
    const f = files[i];
    console.log(`${f.name} (${f.type}, ${f.size} bytes)`);
    i++;
  }
});
```

## 🔢 FileReader 类型
`FileReader`类型表示一种异步文件读取机制。可以把`FileReader`想象成类似于`XMLHttpRequest`，只不过是用于从文件系统读取文件，而不是从服务器读取数据，使用`File`或`Blob`对象指定要读取的文件或数据。

`FileReader`是一个构造函数，所以使用的时候要进行实例化：

```js
const reader = new FileReader();
```

`FileReader`实例化对象的属性：

| `.error` | 表示在读取文件时发生的错误 |
| --- | --- |
| `.readyState` | 表示读取文件的状态（0 未加载数据，1 数据正在被加载，2 已完成全部的读取请求） |
| `.result` | 文件的内容，该属性仅在读取操作完成后才有效，数据的格式取决于使用哪个方法来启动读取操作。 |

`FileReader`实例化对象的读取文件数据的方法：

| `.readAsDataURL(file)` | 读取文件并将内容的数据以 URI 的形式保存在`.result`属性上，`.result`属性中将包含一个`data: URL`格式的`Base64`字符串以表示所读取文件的内容。 |
| --- | --- |
| `.readAsText(file, encoding)` | 从文件中读取纯文本内容并保存在`result`属性中。第二个参数表示编码，是可选的。 |
| `.readAsArrayBuffer(file)` | 读取文件并将文件内容以`ArrayBuffer`形式保存在`result`属性。 |
| `.readAsBinaryString(file)` | 读取文件并将每个字符的二进制数据保存在`result`属性。 |
| `.abort()` | 中止读取操作，在返回时，`readyState`属性为 2。 |

因为这些读取方法是异步的，所以每个`FileReader`实例化对象存在相应的事件：

| `abort()` | 该事件在读取操作被中断时触发。 |
| --- | --- |
| `error()` | 该事件在读取操作发生错误时触发，触发`error`事件时`error`属性会包含错误信息，属性的`code`表示错误类型：1（未找到文件）、2（安全错误）、3（读取被中断）、4（文件不可读）或 5（编码错误） |
| `load()` | 该事件在读取操作完成时触发。 |
| `loadstart()` | 该事件在读取操作开始时触发。 |
| `loadend()` | 该事件在读取操作结束时（要么成功，要么失败）触发。 |
| `progress()` | 该事件在读取`Blob`时触发，每 50 毫秒就会触发一次，其与`XHR`的`progress`事件具有相同的信息：`lengthComputable`、`loaded`和`total`。 |

🌰 上传文件案例：

```js
let filesList = document.getElementById("fileInput");

filesList.addEventListener("change", (event) => {

  let info = "",
    	output = document.getElementById("output"),
    	progress = document.getElementById("progress"),
    	files = event.target.files,
    	type = "default",
    	reader = new FileReader();

  // 如果文件类型是图片
  if (/image/.test(files[0].type)) {
    reader.readAsDataURL(files[0]);
    type = "image";
  } else {
    reader.readAsText(files[0]);
    type = "text";
  }

  // 发生错误
  reader.onerror = function() {
    output.innerHTML = "无法读取文件，错误码是：" + reader.error.code;
  };

  // 上次进度
  reader.onprogress = function(event) {
    if (event.lengthComputable) {
      progress.innerHTML = `${event.loaded}/${event.total}`;
    }
  };

  // 上传成功
  reader.onload = function() {
    let html = "";
    switch(type) {
      case "image":
        html = `<img src="${reader.result}">`;
        break;
      case "text":
        html = reader.result;
        break;
    }

    output.innerHTML = html;
  };

});
```

## 🔢 FileReaderSync 类型
顾名思义，`FileReaderSync`类型就是`FileReader`的同步版本。

这个类型拥有与`FileReader`相同的方法，只有在整个文件都加载到内存之后才会继续执行。`FileReaderSync`只在工作线程中可用，因为如果读取整个文件耗时太长则会影响全局。

## 🔢 Blob 与 File 截取
某些情况下，可能需要读取部分文件而不是整个文件。为此，`File`对象提供了一个名为`slice()`的方法。

`slice()`方法接收两个参数：起始字节、要读取的字节数。该方法返回一个`Blob`的实例，而`Blob`实际上是`File`的超类。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1668492164948-c6db5411-4b1f-4a73-b5ef-5b7741e9155d.png)

`Blob`表示二进制大对象，是 JavaScript 对不可修改二进制数据的封装类型。包含字符串的数组、`ArrayBuffers`、`ArrayBufferViews`，甚至其他`Blob`都可以用来创建`Blob`。

`Blob`构造函数可以接收一个`options`参数，并在其中指定 MIME 类型：

```js
console.log(new Blob(['foo']));
// Blob {size: 3, type: ""}

console.log(new Blob(['{"a": "b"}'], { type: 'application/json' }));
// {size: 10, type: "application/json"}

console.log(new Blob(['<p>Foo</p>', '<p>Bar</p>'], { type: 'text/html' }));
// {size: 20, type: "text/html"}
```

`Blob`对象有一个`size`属性和一个`type`属性，还有一个`slice()`方法用于进一步切分数据。另外也可以使用`FileReader`从`Blob`中读取数据。

```js
let filesList = document.getElementById("fileInput");

filesList.addEventListener("change", (event) => {

  let info = "",
    	output = document.getElementById("output"),
    	progress = document.getElementById("progress"),
    	file = event.target.files[0],
    	reader = new FileReader(),
    	blob = file.slice(0, 32);

  if (blob) {
    reader.readAsText(blob);

    // 读取失败
    reader.onerror = function() {
      output.innerHTML = "无法读取文件，错误代码为：" + reader.error.code;
    };

    // 读取成功
    reader.onload = function() {
      output.innerHTML = reader.result;
    };

  } else {
    console.log("您的浏览器不支持 slice()");
  }

});
```

只读取部分文件可以节省时间，特别是在只需要数据特定部分比如文件头的时候。

## 🔢 对象 URL 与 Blob
对象 URL 有时候也称作 Blob URL，是指引用存储在`File`或`Blob`中数据的 URL。

对象 URL 的优点是不用把文件内容读取到 JavaScript 也可以使用文件，只要在适当位置提供对象 URL 即可。

要创建对象 URL，可以使用`window.URL.createObjectURL()`方法并传入`File`或`Blob`对象。

这个函数返回的值是一个指向内存中地址的字符串。因为这个字符串是 URL，所以可以在`DOM`中直接使用。

```js
let filesList = document.getElementById("fileInput");

filesList.addEventListener("change", (event) => {

  let info = "",
    	output = document.getElementById("output"),
    	progress = document.getElementById("progress"),
    	files = event.target.files, reader = new FileReader(),
    	url = window.URL.createObjectURL(files[0]);

  if (url) {
    if (/image/.test(files[0].type)) {
      output.innerHTML = `<img src="${url}">`;
    } else {
      output.innerHTML = "Not an image.";
    }
  } else {
    output.innerHTML = "Your browser doesn't support object URLs.";
  }
});
```

如果把对象 URL 直接放到`<img>`标签，就不需要把数据先读到 JavaScript 中了。`<img>`标签可以直接从相应的内存位置把数据读取到页面上。

如果想表明不再使用某个对象 URL，则可以把它传给`window.URL.revokeObjectURL()`。

页面卸载时，所有对象 URL 占用的内存都会被释放。

## 🔢 Blob 与 File 互转
`File`对象转化为`Blob`对象：

```js
const file = new File(["Hello, world!"], "hello.txt", { type: "text/plain" });
const blob = new Blob([file], { type: file.type });
console.log(blob);
```

例如从文件输入转换`File`为`Blob`的示例：

```js
<input type="file" id="fileInput">

<script>
  document.getElementById('fileInput').addEventListener('change', function(event) {
    const file = event.target.files[0]; // 获取第一个文件
    const blob = new Blob([file], { type: file.type });

    const reader = new FileReader();
    reader.onload = function(e) {
      console.log(e.target.result); // 输出文件内容
    };
    reader.readAsText(blob); // 读取 Blob 内容
  });
</script>
```

`Blob`对象转化为`File`对象：

```js
const blob = new Blob(["Hello, world!"], { type: "text/plain" });
const file = new File([blob], "hello.txt", { type: blob.type, lastModified: Date.now() });
console.log(file);
```

例如从`Blob`创建并下载`File`示例：

```js
<button id="downloadBtn">Download Blob as File</button>

<script>
  document.getElementById('downloadBtn').addEventListener('click', function() {
    const blob = new Blob(["Hello, world!"], { type: "text/plain" });
    const file = new File([blob], "hello.txt", { type: blob.type, lastModified: Date.now() });

    const url = URL.createObjectURL(file);
    const a = document.createElement('a');
    a.href = url;
    a.download = file.name;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url); // 释放 URL 对象
  });
</script>
```

## 🔢 读取拖放文件
组合使用 HTML5 拖放 API 与 File API 可以创建读取文件信息的有趣功能。

在页面上创建放置目标后，可以从桌面上把文件拖动并放到放置目标。这样会像拖放图片或链接一样触发`drop`事件。

被放置的文件可以通过事件的`event.dataTransfer.files`属性读取，这个属性保存着一组`File`对象，就像文本输入字段一样。

```js
let droptarget = document.getElementById("droptarget");

function handleEvent(event) {
  let info = "",
    output = document.getElementById("output"),
    files,
    i,
    len;

  event.preventDefault();

  if (event.type == "drop") {
    files = event.dataTransfer.files;
    i = 0;
    len = files.length;

    while (i < len) {
      info += `${files[i].name} (${files[i].type}, ${files[i].size} bytes)<br>`;
      i++;
    }

    output.innerHTML = info;
  }
}

droptarget.addEventListener("dragenter", handleEvent);
droptarget.addEventListener("dragover", handleEvent);
droptarget.addEventListener("drop", handleEvent);
```

在`drop`事件处理程序中，可以通过`event.dataTransfer.files`读到文件，此时可以获取文件的相关信息。
