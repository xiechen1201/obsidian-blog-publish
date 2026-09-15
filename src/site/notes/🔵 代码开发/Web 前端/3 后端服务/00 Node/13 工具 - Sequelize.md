---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/3 后端服务/00 Node/13 工具 - Sequelize/","dg-note-properties":{}}
---

---
---
Github：

[GitHub - sequelize/sequelize: Feature-rich ORM for modern Node.js and TypeScript, it supports PostgreSQL (with JSON and JSONB support), MySQL, MariaDB, SQLite, MS SQL Server, Snowflake, Oracle DB (v6), DB2 and DB2 for IBM i.](https://github.com/sequelize/sequelize)

[GitHub - demopark/sequelize-docs-Zh-CN at v6](https://github.com/demopark/sequelize-docs-Zh-CN/tree/v6)

## 🔢 ORM 工具
今天介绍一个 ORM 工具 -- Sequelize，它既支持 JS 也支持 TS，算是比较成熟的 ORM 工具了。

什么是 ORM 工具？

ORM（Object Relational Mapping）对象关系映射，它可以在我们使用 Node 开发服务端的时候更加优雅、安全的操作数据库。

ORM 会隐藏具体的数据库底层细节，他把数据库的表和字段，映射成为 JavaScript 中的类和对象，让开发者直接操作 JS 来和数据库进行交互，最终完成对不同数据库的操作，而不再需要编写大量的 SQL 语句。

使用 ORM 工具的优势：

- 开发者不再需要关心数据库，仅需要关心 JavaScript 对象；
- 可以轻易的完成对数据库的移植工作；
    - 例如最开始使用的数据库是 MySQL，但是后续因为某些原因可能需要切换成其他数据库；
- 无需编写、拼接复杂的 SQL 语句就可以完成增删改查，通过 ORM 的 API 操作数据库；
    - ORM 会自动编写适合的 SQL 语句操作数据；

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1744967412675-f2608674-3cee-44d5-8900-ab0aa965c8a9.png)

## 🔢 基础使用
这里我举几个例子简单演示一下。

首先要安装 Sequelize 工具和对应的数据库驱动程序：

```bash
$ npm install --save sequelize
$ npm install --save mysql2 # 我选择的是 MySQL
```

然后创建一个文件，用于连接数据库：

```js
/* modules/db.js */

const { Sequelize } = require("sequelize");

// 新建对象，传入配置
// 分别是：数据库名称、账户名、密码、其他配置
const sequelize = new Sequelize("orm_myschool_db_2", "root", "xxxxxx", {
    host: "localhost", // 服务地址
    dialect: "mysql", // 数据库的类型
});

module.exports = sequelize;
```

这样我们就得到了一个`sequelize`的模型实例对象，它表示与一个数据库的连接。

<br/>tips
❕信息

需要提前创建好一个名为“orm_myschool_db_2”的数据库。

<br/>

## 🔢 定义表模型
接着，我们就可以定义模型了，模型是 Sequelize 的本质，是对数据库表的抽象概念，可以理解一个模型就是一张表。

```js
/* modules/admin.js */

const sequelize = require("./db.js"); // 导入刚刚写的连接实例
const { DataTypes } = require("sequelize"); // 导入数据类型

// 定义模型（首字母大写）
const Admin = sequelize.define(
    "Admin", // 模型的名称
    {
        // 不需要配置 ID 主键，因为会自动生成
        loginId: { // 表字段
            type: DataTypes.STRING, // 字段类型
            allowNull: false // 允许为空
        },
        loginPwd: {
            type: DataTypes.STRING,
            allowNull: false
        },
        name: {
            type: DataTypes.STRING,
            allowNull: false
        }
    },
    {
      tableName: "admin" // 强制指定表的名字为“admin”，否则会变成 Admin 复数形式 Admins
    }
);
```

目前，我们只是连接了数据库并且定义了模型（表），但是还没有创建数据库和表。

我们只需要调用一下`sync`方法即可：

```js
Admin.sync({ alter: true }).then(() => {
    console.log("Admin 表同步成功");
});
```

最后只要执行代码即可：

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1744968704653-185cd63b-d23d-4266-9566-fbe64511c59a.png)

然后我们查看 Navicat 工具就可以看到这个表已经被创建成功了！

![](/img/user/%F0%9F%94%B5%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/3%20%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1/00%20Node/_assets/1744968816085-36ec2079-9380-4281-be41-cc9ba349df1e.png)

（默认情况下，Sequelize 使用数据类型`DataTypes.DATE`自动向每个模型添加`createdAt`和`updatedAt`字段。这些字段会自动进行管理，每当使用 Sequelize 创建或更新内容时，这些字段都会被自动设置。`createdAt`字段将包含代表创建时刻的时间戳，而`updatedAt`字段将包含最新更新的时间戳。）

## 🔢 操作表数据
最后再介绍一下如何操作表的数据。

当我们通过`sequelize.define`定义一个模型后，会得到一个模型的类（例如`Admin`模型类），这个时候我们需要使用类的`build()`方法再得到一个模型的实例对象（类比为表的行数据），该对象就可以操作表的数据啦：

```js
// 构建模型实例
const instance = Admin.build({
    loginId: "admin",
    loginPwd: "admin",
    name: "超级管理员"
});
```

但是`build()`方法仅仅是创建对象，该对象表示可以映射到数据库的数据，为了将这个实例对象真正的保存到数据库，还需要调用一下`save()`方法：

```js
await instance.save();
```

或者可以直接使用`create()`方法代替`build()`+`save()`方法：

```js
const instance = Admin.create({
    loginId: "admin",
    loginPwd: "admin",
    name: "超级管理员"
})
console.log(instance instanceof Admin); // true
```

如果想要删除数据只需要调用`destroy()`方法：

```js
instance.destroy(); // 删除
```
