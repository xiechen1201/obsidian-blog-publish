---
{"dg-publish":true,"permalink":"//web/3/00-node/12-mysql2/","dg-note-properties":{}}
---

Github：

[GitHub - sidorares/node-mysql2: :zap: fast mysqljs/mysql compatible mysql driver for node.js](https://github.com/sidorares/node-mysql2)

mysql2 是适用于 Node 的 MySQL 驱动程序，让 Node 拥有了操作数据库的能力，它可以跨平台使用的。

什么是驱动程序？

驱动程序是连接内存和其他存储介质之间的桥梁，因此 mysql2 是连接内存数据和 MySQL 数据的桥梁。

## 🔢 基础使用
关于如何安装 mysql2 就不展开叙述了，文档中写的很清楚。

这里简单描述一下如何使用，安装 mysql2 驱动程序后首先需要导入它并创建一个数据库连接：

```js
import mysql from "mysql2/promise";

// 创建一个数据库连接
const connection = await mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "00000000",
    database: "yuanLaoShiTestDB"
});
```

然后我们就得到了一个`connection`对象，接着我们就可以使用`connection`对象操作 SQL 语句了：

```js
// 查询数据
try {
  const [results, fields] = await connection.query(
    'SELECT * FROM `table` WHERE `name` = "Page" AND `age` > 45'
  );
  console.log(results); // 结果集
  console.log(fields); // 额外的元数据（如果有的话）
} catch (err) {
  console.log(err);
}
```

对于增删改数据都是一样的：

```js
// 增加数据
try {
    const result = await connection.query(
        "INSERT INTO `company` (`name`,`location`,`buildDate`) VALUES ('abc','这是一个测试地址',curdate())"
    );
    console.log(result);
} catch (error) {
    console.log(err);
}

// 修改数据
try {
    const result = await connection.query(
        "UPDATE `company` SET `name`='bcd' WHERE `id`=4"
    );
    console.log(result);
} catch (error) {
    console.log(err);
}

// 删除数据
try {
    const result = await connection.query(
        "DELETE FROM `company` WHERE `id`=4"
    );
    console.log(result);
} catch (error) {
    console.log(err);
}
```

最后使用`end()`方法断开数据库连接：

```js
// 断开数据库连接
connection.end();
```

## 🔢 防 SQL 注入
什么是 SQL 注入？

SQL 注入是指用户通过注入 SQL 语句的方式到最终查询中，导致了整个 SQL 与预期的行为不符合。

假设查询用户名的信息是从前端传来的：

```js
const name = req.query.name;

try {
    const [results, fields] = await connection.query(
        `SELECT * FROM company WHERE name = '${name}'`
    );
    console.log(results);
} catch (err) {
    console.log(err);
}
```

但如果用户输入的是一个 SQL 语句呢？

```js
' OR '1'='1
```

因此，实际的查询语句就变成了这样：

```plsql
SELECT * FROM company WHERE name = '' OR '1'='1'
```

这样`OR`条件就永远成立，直接返回了数据库中所有的数据。

因此我们不能直接使用变量作为 SQL 的子语句。

mysql2 提供了`execute()`方法用于查询语句，和`query()`不同的是`execute()`可以使用`?`对子语句进行占位，最后进行替换。

```js
const name = req.query.name;

try {
    const [results, fields] = await connection.execute(
        "SELECT * FROM company WHERE name = ?",
        [name]
    );
    console.log(results);
} catch (err) {
    console.log(err);
}
```

这是因为`execute()`的工作方式是先将 SQL 进行预编译，提取占位参数，最后再把占位参数替换掉。而这个占位的参数又被进行类型检查、转义处理和分离逻辑，即使是 SQL 语句也会被当成普通字符串处理，这样就避免了 SQL 注入。

如果想要实现模糊查询也是没问题的，只需要将`%%`提取到外面即可：

```js
const [results, fields] = await connection.execute(
  "SELECT * FROM `employee` WHERE `name` LIKE ?",
  ["谢"]
);
```

## 🔢 连接池
在不使用连接池且并发比较多的情况下，例如网站有 100 个用户同时访问，如果创建 100 个连接会把数据库压垮，这个时候就可以使用连接池。

连接池就好像是数据库连接的共享仓库，仓库里面提取准备好了一些连接，程序需要的时候就借用一个，用完就还回去，而不是每次在程序需要的时候创建一个新的连接。

```js
const pool = await mysql.createPool(...);

// 使用连接池查询
const [rows] = await pool.query('SELECT * FROM users');
```

使用连接池查询和普通查询差不多，但是背后却拥有更高效的管理机制。

创建连接池可以配置很多参数，示例：

```js
// 创建连接池，设置连接池的参数
const pool = mysql.createPool({
    host: "localhost",
    user: "root",
    password: "00000000",
    database: "yuanLaoShiTestDB",
    waitForConnections: true, // 等待空闲的连接
    connectionLimit: 10, // 最多能同时创建的连接数上限（不管是不是空闲）
    maxIdle: 10, // 最大空闲连接数，默认等于 `connectionLimit`
    idleTimeout: 60000, // 空闲连接超时，以毫秒为单位，默认值为 60000 ms
    queueLimit: 0, // 连接不够用，队列中的最大等待连接数，默认值为 0，表示不限制
    enableKeepAlive: true,
    keepAliveInitialDelay: 0
});
```

更多详见：

[createPool | Quickstart](https://sidorares.github.io/node-mysql2/zh-CN/docs/examples/connections/create-pool)
