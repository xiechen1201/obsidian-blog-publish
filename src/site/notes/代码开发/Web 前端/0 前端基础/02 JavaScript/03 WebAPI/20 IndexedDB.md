---
{"dg-publish":true,"permalink":"/代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/20 IndexedDB/","dg-note-properties":{}}
---

---
---
IndexedDB 是浏览器存储结构化数据的方案，其背后的思想是提供一套 API，方便 JavaScript 对象的存储和获取，同时也支持查询和搜索。

## 🔢 数据库

IndexedDB 是类似于 MySQL 或 Web SQL Database 的数据库。和传统数据库的区别在于，IndexedDB 使用对象存储来保存数据，而不是使用表格。

使用 IndexedDB 的第一步是调用 `indexedDB.open()` 方法，并传入要打开的数据库名称。如果这个数据库存在，则会发送一个打开请求；否则会创建并打开这个数据库。

该方法返回一个 `IDBRequest` 实例，可以在实例上添加 `onerror` 和 `onsuccess` 事件处理程序。

示例：

```js
let db;
let openRequest;
let version = 1;

// 打开数据库
openRequest = indexedDB.open("admin", version);

openRequest.onerror = (event) => {
  console.log(event.target.error);
};

openRequest.onsuccess = (event) => {
  db = event.target.result;
};
```

> [!tip]
> `open()` 方法中的版本会被转换为一个 unsigned long 数值（一种整数数据类型，常见于 C/C++、系统编程和 Web API 规范中），因此不要使用小数作为版本号。

## 🔢 对象存储

当和数据库建立连接之后，下一步就是使用对象存储。如果数据库的版本和期待的不一致，可能需要创建对象存储。

> [!tip]
> 可以把「对象存储」类比为「数据库的表」。它具有表的能力，但又区别于表。

假如要存储用户的账户信息，可以使用对象表示一条数据：

```js
const user = {
  username: "007",
  firstName: "James",
  lastName: "Bond",
  password: "foo",
};
```

观察这个对象可以看出，`username` 属性非常适合作为键。用户名必须保证全局唯一，创建对象存储时必须指定一个键。

数据库的版本决定了数据库的模式，包括数据库中的对象存储和这些对象存储的结构。

当调用 `open()` 方法时：

- 如果数据库还不存在，则会创建一个新的数据库，然后触发 `upgradeneeded` 事件，并在事件处理中创建数据库模式；
- 如果数据库存在，但是版本不一样，则会触发 `upgradeneeded` 事件，并在事件处理中更新数据库模式；

示例，为上述用户信息创建对象存储：

```js
openRequest.onupgradeneeded = (event) => {
  const db = event.target.result;

  // 如果存在则删除当前 objectStore。测试时可以这样做，
  // 但这样会在每次执行事件处理程序时删除已有数据。
  if (db.objectStoreNames.contains("users")) {
    db.deleteObjectStore("users");
  }

  // 创建对象存储
  db.createObjectStore("users", { keyPath: "username" });
};
```

`keyPath` 属性表示作为键的对象属性名。

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1776932443391-2de4fb97-a418-4222-90bc-2b37a88904e5.png)

## 🔢 事务

创建对象存储之后，剩下的所有操作都是通过「事务」完成的。「事务」要通过调用数据库对象的 `transaction()` 方法创建。

> [!tip]
> 事务是操作对象存储（可以想象为表）的唯一入口。

```js
const transaction = db.transaction();
```

调用方法时如果不传递参数，则表示只具有「读取」权限。更具体的方式是指定一个或多个要访问的对象存储名称。

```js
openRequest.onupgradeneeded = (event) => {
  const db = event.target.result;

  // 如果存在 users 表，先删除（仅测试用）
  if (db.objectStoreNames.contains("users")) {
    db.deleteObjectStore("users");
  }

  // 创建 users 表
  db.createObjectStore("users", { keyPath: "username" });
};
```

如果要访问多个对象存储，可以给方法传递一个字符串数组：

```js
const transaction = db.transaction(["users", "anotherStore"]);
```

目前事务还都是「读取」权限。如果要更改访问模式，则需要传递第二个参数，可选值包括：

- `"readonly"`
- `"readwrite"`
- `"versionchange"`

```js
const transaction = db.transaction("users", "readwrite");
```

这样事务就可以对 `users` 对象存储进行读写了。

有了「事务」对象的引用，就可以使用 `objectStore()` 方法并传入对象存储的名称来访问特定的对象存储。然后使用 `add()` 和 `put()` 添加和更新对象，使用 `get()` 获取对象，使用 `delete()` 删除对象，使用 `clear()` 删除所有对象。

`get()` 和 `delete()` 方法都接收对象的键作为参数，以上几个方法都会创建新的请求对象。

```js
openRequest.onsuccess = (event) => {
  db = event.target.result;

  // 创建一个事务，声明这次事务可以访问 users 这个 object store，并且权限是读写
  const transaction = db.transaction("users", "readwrite");

  // 从这个事务里取出 users 这张“表”的操作入口
  const objectStore = transaction.objectStore("users");
  const request = objectStore.get("007");
};
```

> [!tip]
> `transaction()` 是操作上下文，`objectStore()` 是在这个上下文里拿到具体表的操作对象。

因为一个事务可以完成任意多个请求，所以事务对象本身也有事件处理程序。

```js
transaction.onerror = (event) => {
  // 整个事务被取消
};

transaction.oncomplete = (event) => {
  // 整个事务成功完成
};
```

> [!warning]
> 不能通过「事务」的 `oncomplete` 事件的 `event` 对象来访问 `get()` 请求返回的数据。因此，仍然需要通过 `request` 请求对象的 `onsuccess` 事件来获取数据。

## 🔢 插入对象

在获得对象存储的引用后，就可以调用方法写入数据了。

当使用 `add()` 和 `put()` 写入数据时，如果对象存储中已经存在同名的键，前者会导致错误，后者会简单地重写对象。

每次调用这两个方法时都会创建新的请求对象。如果想要验证是否成功，可以为这个请求绑定事件：

```js
const request = objectStore.add(user);

request.onerror = () => {
  // 处理错误
};

request.onsuccess = () => {
  // 处理成功
};
```

## 🔢 通过游标查询

使用事务可以通过一个已知的键来获取一条记录。如果要获取多条数据，则需要在事务中创建一个「游标」。游标是一个指向结果集的指针。

调用 `openCursor()` 可以创建一个游标，该方法也会返回一个 `request` 对象，可以绑定 `success` 和 `error` 事件。

```js
// 创建一个事务，声明这次事务可以访问 users 这个 object store，并且权限是读写
const transaction = db.transaction("users", "readwrite");

// 从这个事务里取出 users 这张“表”的操作入口
const objectStore = transaction.objectStore("users");
const request = objectStore.openCursor();

request.onsuccess = (event) => {
  console.log("🚀 ~ event:", event.target.result);
};

request.onerror = (event) => {};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1779086658041-998d7e62-f918-44ba-b815-ffc88f30c14e.png)

`event.target.result` 对象的属性表示：

- `direction`，表示游标的前进方向，以及是否应该遍历所有重复的值；
  - `"next"`
  - `"nextunique"`
  - `"prev"`
  - `"prevunique"`
- `key`，对象的键；
- `value`，对象的值；
- `primaryKey`，游标使用的键；

游标还可以调用 `update()` 方法更新某个记录，参数为指定对象，用于替换当前游标的值。调用 `update()` 后也会返回一个新请求。

```js
openRequest.onsuccess = (event) => {
  const db = event.target.result;

  // 创建一个事务，声明这次事务可以访问 users 这个 object store，并且权限是读写
  const transaction = db.transaction("users", "readwrite");

  // 从这个事务里取出 users 这张“表”的操作入口
  const objectStore = transaction.objectStore("users");

  // 创建游标
  const request = objectStore.openCursor();

  request.onsuccess = (event) => {
    console.log("🚀 ~ event:", event.target.result);
    const cursor = event.target.result;

    // 必须进行空值判断
    if (cursor) {
      if (cursor.key === "007") {
        const newValue = { ...cursor.value };
        newValue.password = "newPassword";

        // 更新数据
        const updateRequest = cursor.update(newValue);
        updateRequest.onsuccess = () => {
          console.log("更新成功");
        };
        updateRequest.onerror = (event) => {
          console.log(event.target.error);
        };
      }
    }
  };
};
```

游标还可以调用 `delete()` 方法删除某个记录，同样返回一个请求对象。

> [!warning]
> 如果事务 `transaction` 没有修改对象存储的权限，则 `update()` 和 `delete()` 都会抛出错误。

默认情况下，每个游标只会创建一个请求。要创建另外一个请求，必须调用下面其中一个方法：

- `continue(key)`，移动到结果集的下一条记录，`key` 是可选的，如果提供了 `key`，则表示移动到指定位置；
- `advance(count)`，游标向前移动指定的 `count` 条记录；

这两个方法会让游标重用相同的请求，因此也会重用 `success` 和 `error` 事件。

```js
const request = objectStore.openCursor();

request.onsuccess = (event) => {
  console.log("🚀 ~ event:", event.target.result);
  const cursor = event.target.result;

  // 必须进行空值判断
  if (cursor) {
    // 更新数据
    if (cursor.key === "007") {
      const newValue = { ...cursor.value };
      newValue.password = "newPassword";

      const updateRequest = cursor.update(newValue);
      updateRequest.onsuccess = () => {
        console.log("更新成功");
      };
      updateRequest.onerror = (event) => {
        console.log(event.target.error);
      };
    }

    // 继续下一条数据
    cursor.continue();
  }
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1779088774223-fc232d70-0bef-4bb1-ab88-a731e33a4c09.png)

## 🔢 键范围

使用键范围可以让游标更容易管理。键范围对应 `IDBKeyRange` 的实例，有 4 种方式指定键范围。

> [!tip]
> `IDBKeyRange` 是一个全局对象。

1. 使用 `only()` 方法并传入要获取的键。

```js
const onlyRange = IDBKeyRange.only("007");
```

这种方式保证只获取键为 `"007"` 的值，类似于直接访问对象存储并调用 `get("007")`。

2. 定义结果集的下限，下限表示游标的开始位置。

```js
// 从 "007" 记录开始，直到最后
const lowerRange = IDBKeyRange.lowerBound("007");
```

以上代码表示游标从 `"007"` 这个键开始，直到最后。

如果想要从 `"007"` 后面的记录开始，可以传入第二个参数 `true`：

```js
IDBKeyRange.lowerBound("007", true);
```

3. 定义结果集的上限，指定游标不会越过的记录，也就是结束位置。

```js
// 从头开始，到 "ace" 记录为止
const upperRange = IDBKeyRange.upperBound("ace");
```

如果不想包含指定的键，同样可以传入 `true`：

```js
IDBKeyRange.upperBound("ace", true);
```

4. 如果要同时指定上限和下限，可以使用 `bound()` 方法。

```js
// 从 "007" 的下一条记录开始，到 "ace" 的前一条记录停止
const boundRange = IDBKeyRange.bound("007", "ace", true, true);
```

`bound()` 方法接收 4 个参数，分别是：开始的键、结束的键、不包含开始的键、不包含结束的键。

定义好范围之后，可以把结果传递给 `openCursor()` 方法，这样就可以对范围进行控制。

```js
openRequest.onsuccess = (event) => {
  const db = event.target.result;

  // 创建一个事务，声明这次事务可以访问 users 这个 object store，并且权限是读写
  const transaction = db.transaction("users", "readwrite");

  // 从这个事务里取出 users 这张“表”的操作入口
  const objectStore = transaction.objectStore("users");

  // 从 "007" 的下一条记录开始，到 "ace" 的前一条记录停止
  const boundRange = IDBKeyRange.bound("007", "ace", true, true);
  const request = objectStore.openCursor(boundRange);

  request.onsuccess = (event) => {
    console.log("🚀 ~ event:", event.target.result);
    const cursor = event.target.result;

    // 必须进行空值判断
    if (cursor) {
      if (cursor.key === "007") {
        const newValue = { ...cursor.value };
        newValue.password = "newPassword";

        const updateRequest = cursor.update(newValue);
        updateRequest.onsuccess = () => {
          console.log("更新成功");
        };
        updateRequest.onerror = (event) => {
          console.log(event.target.error);
        };
      }

      cursor.continue();
    }
  };
};
```

![](/img/user/%E4%BB%A3%E7%A0%81%E5%BC%80%E5%8F%91/Web%20%E5%89%8D%E7%AB%AF/0%20%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/02%20JavaScript/03%20WebAPI/_assets/1779103714958-3ab794e3-b84c-4bd2-a30b-6cb155b1a679.png)

不再查询到 `"007"`。

## 🔢 设置游标方向

`openCursor()` 方法实际上可以接收两个参数，一个是 `IDBKeyRange` 实例（游标范围），另外一个是表示方向的字符串。

通常，游标都是从对象存储的第一条开始。当调用 `continue()` 或 `advance()` 时，游标都会向后前进，默认方向就是 `"next"`。

如果对象存储中存在重复的记录，可能需要游标跳过重复的内容，可以给 `openCursor()` 传入 `"nextunique"`：

```js
openRequest.onsuccess = (event) => {
  const db = event.target.result;

  // 创建一个事务，声明这次事务可以访问 users 这个 object store，并且权限是读写
  const transaction = db.transaction("users", "readwrite");

  // 从这个事务里取出 users 这张“表”的操作入口
  const objectStore = transaction.objectStore("users");

  // 从 "007" 的下一条记录开始，到 "ace" 的前一条记录停止
  const boundRange = IDBKeyRange.bound("007", "ace", true, true);
  const request = objectStore.openCursor(boundRange, "nextunique");
};
```

也可以反向移动游标，从最后一项向第一项移动。此时需要向 `openCursor()` 传入 `"prev"` 或者 `"prevunique"` 作为第二个参数：

```js
objectStore.openCursor(boundRange, "prevunique");
```

## 🔢 索引

对于某些数据集，可能需要给对象存储指定多个 key。如果同时记录了用户 ID 和用户名，那么可能需要通过任何一种方式来获取用户数据。

> [!info]
> 本意就是说：“除了主键之外，我还能不能用别的字段查数据？”

索引需要在 `onupgradeneeded` 版本变更事务中创建，不能在普通读写事务里创建。

```js
openRequest.onupgradeneeded = (event) => {
  const db = event.target.result;
  const objectStore = db.createObjectStore("users", { keyPath: "username" });

  // 创建索引
  objectStore.createIndex("username", "username", { unique: true });
};
```

`createIndex()` 的参数：

- 索引的名称；
- 索引属性的名称；
- 包含 `unique` 的选项对象，`unique` 表示这个 key 在所有记录中是否唯一；

该方法返回的是 `IDBIndex` 实例对象，在对象存储上调用 `index()` 可以得到同一个实例。

## 🔢 并发问题

IndexedDB 虽然是网页中的异步 API，但是依然存在并发问题。

如果两个不同的浏览器 Tab 页同时打开了同一个网页，则可能出现一个 Tab 页尝试升级数据库，而另外一个 Tab 页尚未就绪的情况。有问题的操作是设置数据库为新版本，而版本变化只能在浏览器只有一个 Tab 页使用数据库的时候完成。

所以，监听 `onversionchange` 事件非常重要。当另外一个同源页面将数据库版本进行升级时，会执行这个回调，最好的处理方式是立即关闭数据库连接，然后等待升级完成。

```js
// 打开数据库
const openRequest = indexedDB.open("admin", version);

openRequest.onsuccess = (event) => {
  const database = event.target.result;
  database.onversionchange = () => database.close();
};
```

这样就可以更好地处理 IndexedDB 相关的并发问题。

## 🔢 限制

IndexedDB 和 Web Storage 一样，数据库和页面同源绑定，所以信息不能跨域。

其次，每个源可以存储的空间比 Web Storage 大很多。Chromium 系浏览器允许所有域使用 80% 的磁盘空间，单个域使用 60% 的磁盘空间（每个浏览器不同）。

## 🔢 包装库

很少有程序员直接操作 IndexedDB API。像 Dexie.js 这样的包装库可以简化很多 JavaScript 存储 API 过程，提供更高效的接口和存储 API 进行交互，使用起来更加方便。

```js
// 创建数据库
const db = new Dexie("MyDatabase");

db.version(1).stores({
  friends: "++id, name, age, *tags",
  gameSessions: "id, score",
});

// 添加对象
await db.friends.add({ name: "Josephine", age: 21 });

// 查询对象
const someFriends = await db.friends
  .where("age")
  .between(20, 25)
  .offset(150)
  .limit(25)
  .toArray();
```
