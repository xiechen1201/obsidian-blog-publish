---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/1 工程化体系/02 Webpack4/00 Webpack 的核心功能/09 Plugin/","dg-note-properties":{}}
---

---
---
前面学习的 Loader 的定位是转换代码，当有一些其他操作的时候 Loader 就无能为力了，例如：

- 当 Webpack 生成文件的时候，顺便多生成一个说明的描述文件
- 当 Webpack 启动编译的时候，控制台输出一句话表示 Webpack 启动成功了
- 当 Webpack 开始编译前，把上一次生成的 dist 目录删除掉

而这种类似的功能就需要借助 Webpack 的另外一个概念 Plugin！Plugin 可以简单的理解为在 Webpack 某个事件触发后干什么事情。

## 🔢 Plugin 的基本写法
回顾前面前面学习过的 Webpack 编译过程，编译过程分为 3 步：初始化、编译、输出。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694571607760-760ef42d-cda3-428b-b61d-c820b9657a60.png)

Webpack 提供了很多的事件 API，例如上面图片中红点都表示一个事件（并不是真实的事件，只作为展示），然后 Plugin 内部的事件监听函数就可以在这些事件触发的时候被执行。

Plugin 本质上就是一个带有`apply`函数的对象：

```js
var plugin = {
    apply: function (compiler) {
        // 函数内注册事件
    }
};

module.exports = plugin;
```

通常，Plugin 会把对象改写为构造函数的模式：

```js
class MyPlugin{
  apply(compiler){

  }
}

var plugin = new MyPlugin();
```

最后在 webpack.config.js 中调用：

```js
module.exports = {
    plugins:[
        new MyPlugin()
    ]
}
```

## 🔢 Compiler && Compilation 对象
`apply`函数会在初始化阶段创建一个`compiler`对象后执行。

`compiler`对象是在初始阶段被创建的，整个 Webpack 打包的过程中只有一个`compiler`对象，后续完成打包工作的是`compiler`对象内部创建的`compilation`。

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694573613058-724c0eda-e7af-43ab-9951-6dd83204cca0.png)

因为整个 Webpack 编译的过程中只会有一个`compiler`对象，所以当 Webpack 开启了 watch 监听后，不会再创建`compiler`，是创建一个新的`compilation`对象！

例如：

```js
class MyPlugin {
    apply(compiler) {
        console.log("MyPlugin 被执行了");
    }
}

module.exports = MyPlugin;
```

```js
const MyPlugin = require("./plugins/MyPlugin");

module.exports = {
    mode: "development",
    watch: true,
    plugins: [new MyPlugin()]
};
```

```bash
# 运行 Webpack
$ npx webpack
```

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694574917259-24be420c-23c4-47e4-b630-1fe004d3dcaa.png)

可以看到`console.log("MyPlugin 被执行了");`只会被执行一次！

## 🔢 事件
`compiler`对象提供了大量的钩子函数（`hooks`，可以理解为事件），开发者可以注册这些钩子函数，参与 Webpack 编译和生成。

可以在`apply`方法中使用下面的代码注册钩子函数：

```js
class MyPlugin{
  apply(compiler){
    compiler.hooks.事件名称.事件类型(name, function(compilation){
      //事件处理函数
    })
  }
}
```

1、事件名称

也就是要被监听的事件名，或者钩子名，所有的钩子事件详见：

[compiler 钩子](https://v4.webpack.docschina.org/api/compiler-hooks/)

2、事件类型

这一部分使用的是 Tapable API，这个小型的库是一个专门用于钩子函数监听的库。

它提供了一些事件类型：

- tap：注册一个同步的钩子函数，函数运行完毕则表示事件处理结束
- tapAsync：注册一个基于回调的异步的钩子函数，函数通过调用一个回调表示事件处理结束
- tapPromise：注册一个基于Promise的异步的钩子函数，函数通过返回的Promise进入已决状态表示事件处理结束

事件类型主要是告诉 Webpack 事件处理函数有没有处理完成，用哪种方式告诉 Webpack。

3、name 属性可以自己命名，主要是方便进行调试

例如：

```js
class MyPlugin {
  apply(compiler) {

    // tap 的方式
    compiler.hooks.compile.tap('MyPlugin', params => {
      console.log('以同步方式触及 compile 钩子。');
    })

    // tapAsync 的方式
    compiler.hooks.run.tapAsync('MyPlugin', (source, target, routesList, callback) => {
      console.log('以异步方式触及 run 钩子。');
      callback();
    });

    // tapPromise 的方式
    compiler.hooks.run.tapPromise('MyPlugin', (source, target, routesList) => {
      return new Promise(resolve => setTimeout(resolve, 1000)).then(() => {
        console.log('以具有延迟的异步方式触及 run 钩子。');
      });
    });

  }
}

module.exports = MyPlugin;
```

处理函数有一个事件参数`compilation`：

```js
class MyPlugin {
    apply(compiler) {
        compiler.hooks.done.tap("MyPlugin", compilation => {})
    }
}
```

`compilation`也可以注册钩子事件：

```js
class MyPlugin {
  apply(compiler) {
    compiler.hooks.beforRun.tap("MyPluginTest", function (compilation) {
      compilation.hooks.xxx.xxx("xxx", function () {});
    });
  }
}
```

更多`compilation`事件详见：

[compilation 钩子](https://v4.webpack.docschina.org/api/compilation-hooks/)

## 🔢 案例
例如我们想要实现一个插件，插件的功能就是在 Webpack 打包完成后多出一个文件，里面记录了文件 dist 目录下的文件名称和文件的大小。

新建一个 plugins/FileListPlugin.js 文件，编写我们的代码：

```js
class FileListPlugin{
  apply(compuler){
    // emit 表示生成资源到 output 目录之前的事件
    compuler.hooks.emit.tap('FileListPlugin',(compilation)=>{
      // compilation 对象有一个 assets 属性，表示所有 chunk 生产的资源
      console.log(compilation.assets)
    })
  }
}

module.exports = FileListPlugin;
```

然后在配置文件中导入使用：

```js
const FileListPlugin = require("./plugins/FileListPlugin");

module.exports = {
    mode: "development",
    devtool: "source-map",
    plugins: [new FileListPlugin()]
};

```

最后运行：

```bash
$ npx webpack
```

结果如下：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694584449170-4a546e79-38a3-418c-b7ae-325c58a8107c.png)

可以看到`assets`属性就是一个对象，里面包含了所有的资源。

接下来我们就可以仿照这个对象添加一个属性：

```js
class FileListPlugin {
  // 如果 new FileListPlugin() 传递了文件名就使用传递的文件名称
  constructor(filename = 'filelist.txt'){
    this.filename = filename
  }

  apply(compuler) {
    compuler.hooks.emit.tap("FileListPlugin", (compilation) => {
      // 创建一个空数组
      var fileList = [];

      // 遍历 assets 对象，key 就是 'main.js' 或者 'main.js.map'
      for (const key in compilation.assets) {
        // 拼接内容
        var content = `【${key}】
                大小：${compilation.assets[key].size() / 1000}KB`;

        fileList.push(content);
      }

      // 把数组转换为字符串
      var str = fileList.join("\n");
      // 往 assets 里面添加一个对象
      // filename 是在 constructor 里面接受的
      compilation.assets[this.filename] = {
        source() {
          return str;
        },
        size() {
          return str.length;
        }
      };
    });
  }
}

module.exports = FileListPlugin;
```

最后查看编译结果：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694584978120-24db1985-5fa9-4491-ab32-4760d159fd44.png)

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/1%20%E5%B7%A5%E7%A8%8B%E5%8C%96%E4%BD%93%E7%B3%BB/02%20Webpack4/00%20Webpack%20%E7%9A%84%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/_assets/1694585031469-bfb11147-3d11-4228-9a06-d6ab8417ca8d.png)

到这里，就实现了一个简单的 Plugin。
