---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/3 后端服务/01 数据库/01 MySQL/","dg-note-properties":{}}
---

承接上一篇文章我们知道 MySQL 是一个关系型数据库，可以很好的表达复杂的数据关系。MySQL 是瑞典 MySQL AB 公司开发的，后来被 Oracle 收购。

MySQL 具有开源、轻量、快速的特点，其也是目前主流的关系型数据库。

## 下载 & 安装
官方下载地址：

[fw_error_www](https://dev.mysql.com/downloads/mysql/)

我的是 Mac 电脑，所以下载后得到的是一个 .dmg 后缀的安装包，根据提示操作安装即可。

<u>不过在安装的过程中，需要设置一个 8 位数据库的 root 账户的密码，这个密码需要你牢记。</u>

<u></u>

## 使用
安装好 MySQL 后，我们要操作它需要使用 CLI 命令。

1、进入 mysql 命令交互。

```bash
$ mysql -uroot -p
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744250237745-c0150ed9-889c-4812-9301-ecf05ccb80cc.png)

可以看到，当登录成功后，控制台打印出了 MySQL 的一些说明信息。接着我们在 mysql 控制台下执行其他命令即可。

2、查看当前数据库中与字符集相关的系统变量。

```bash
$ show variables like 'character\_set\_%';
```

<br/>warning
⚠️ 注意

在 MySQL CLI 执行其他命令必须添加分号`;`。

<br/>

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744250767535-00824241-ae88-4fc3-9d1d-4a57bdb2144e.png)

变量名的释义：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744251482425-10cb24ad-1325-43b5-bb0b-b1dca239d474.png)

3、查看 MySQL 状态；

```bash
$ status;
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744251812872-f506b3e5-1efd-45bf-920e-3d3183a1fc8b.png)

4、查看当前拥有的数据库：

```bash
$ show databases;
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744251888550-a6ecbc80-4954-4066-bf97-4ac591f706f0.png)

展示的结果中，TestDB 和 yuanLaoShiTestDB 都是我后来新建的，默认是没有这两个数据库的。

## Navicat
通过直接使用 CLI 的方式操作 MySQL 还是比较麻烦，我们可以借助一个数据库管理软件帮我们简化操作。

Navicat 是一个非常强大的数据库管理工具，它支持多种主流的数据库。

不过，Navicat 是收费的，不过官方也提供了 Lite 版本供开发者使用基础的功能，因此我使用的也是这个版本。

Lite 版本：

[Navicat | 免费下载 Navicat Premium Lite](https://www.navicat.com.cn/download/navicat-premium-lite)

完整版本：

[Navicat | 产品](https://www.navicat.com.cn/products)

（如果确实想要使用完整版本，且不愿意花钱的同学，可以上网找找破解版本）

安装完成后打开页面就是这样子：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744252621089-272ea7e5-7d6f-483c-9cca-34b42a03b839.png)

然后我们需要让 Navicat 连接我们本地的数据库：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744252671922-599e2b01-b11c-4b41-81c1-eff83f221418.png)

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744252765759-98bbff8d-2795-4fb0-9a01-ebdeb5a2571a.png)

然后就可以看到我们本地的数据库了：

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/01%20%E6%95%B0%E6%8D%AE%E5%BA%93/_assets/1744252799352-24758875-f455-4fb9-9c18-39c3814ae641.png)

（和之前说的一样 TestDB 和 yuanLaoShiTestDB 都是我后面新建的，默认是没有的）
