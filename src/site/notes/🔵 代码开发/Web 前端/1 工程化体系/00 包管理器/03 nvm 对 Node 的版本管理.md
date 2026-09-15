---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/1 工程化体系/00 包管理器/03 nvm 对 Node 的版本管理/","dg-note-properties":{}}
---

---
---
nvm 是一个用于管理 Node 版本的工具。

在实际的开发中，可能会出现多个项目分别使用的是不同的 Node 版本，在这种场景下，管理不同的 Node 版本就显得尤为重要。

nvm 就是用于切换版本的一个工具。

[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

安装完成后就可以使用 nvm 的 CLI 命令。

下面是 nvm 常用的命令：

- 查看 nvm 信息：

```bash
# 查看 nvm 的版本
$ nvm --version

# 查看 nvm 的当前配置
$ nvm current
```

- 安装与卸载：

```bash
# 安装 Node.js 的特定版本
$ nvm install 版本号
# 例如按照 14.5 的版本
$ nvm install 14.15.0
# 查看当前安装的 Node.js 版本
$ node -v

# 卸载 Node.js 的特定版本
$ nvm uninstall 版本号
```

- 设置默认的 Node 版本：

```bash
# 设置默认的 Node.js 版本
$ nvm alias default 版本号
```

- 切换 Node 版本：

```bash
# 切换当前使用的 Node.js 版本
$ nvm use 版本号
# 例如，切换到版本 14.15.0
$ nvm use 14.15.0
```

- 列出 Node 的版本：

```bash
# 列出已安装的 Node.js 版本
$ nvm ls

# 列出所有可用的 Node.js 版本（包括本地和远程）
$ nvm ls-remote
```

- 使用指定版本的 Node 运行命令：

```bash
# 运行某个 Node.js 版本的 npm
$ nvm use 版本号 --silent && npm <command>
# 例如，使用 Node.js 版本 14.15.0 运行 npm list
$ nvm use 14.15.0 --silent && npm list
```

这些命令可以帮助我们更灵活地管理 Node.js 环境，特别是在需要为不同的项目使用不同版本的 Node.js 时。
