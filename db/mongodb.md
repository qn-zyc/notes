<!-- TOC -->

- [1. install](#1-install)
    - [1.1. 在 mac os 上安装](#11-在-mac-os-上安装)
- [2. Run MongoDB](#2-run-mongodb)
- [3. CRUD 操作](#3-crud-操作)
    - [3.1. Insert Documents](#31-insert-documents)
        - [3.1.1. 插入单个文档](#311-插入单个文档)
        - [3.1.2. 插入多个文档](#312-插入多个文档)
    - [3.2. Query Documents](#32-query-documents)
        - [3.2.1. 查询 collection 中所有文档](#321-查询-collection-中所有文档)
        - [3.2.2. 指定等式条件](#322-指定等式条件)
        - [3.2.3. 使用查询操作符](#323-使用查询操作符)
        - [3.2.4. AND 多个条件](#324-and-多个条件)
        - [3.2.5. OR 多个条件](#325-or-多个条件)
        - [3.2.6. 混合 AND 和 OR](#326-混合-and-和-or)
    - [3.3. Update Documents](#33-update-documents)
        - [update](#update)
        - [3.3.1. updateOne](#331-updateone)
        - [3.3.2. updateMany](#332-updatemany)
        - [3.3.3. replaceOne](#333-replaceone)
        - [3.3.4. Upsert Option](#334-upsert-option)
- [4. 参考](#4-参考)

<!-- /TOC -->


# 1. install

* [下载](https://www.mongodb.com/download-center?jmp=nav#community)
* [安装文档](https://docs.mongodb.com/manual/installation/)

## 1.1. 在 mac os 上安装

1. 下载二进制包: `curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.5.tgz`
2. 解压: `tar -zxvf mongodb-osx-x86_64-3.4.5.tgz`
3. 配置环境变量 PATH，路径是解压后的目录中的 bin 目录：`export PATH=$PATH:/mongoDB/mongodb-osx-x86_64-3.4.5/bin`


# 2. Run MongoDB

1. 创建 data 目录

    MongoDB 默认使用 `/data/db`，如果使用其他的目录，在启动 mongod 进程时使用 `dbpath` 选项指定。

2. 设置 data 目录的权限

    需要读、写权限。

3. 启动

    ```bash
    mongod                                    # 使用默认 data 目录
    mongod --dbpath <path to data directory>  # 指定 data 目录
    ```

4. 连接

    ```bash
    mongo
    ```


# 3. CRUD 操作

## 3.1. Insert Documents

* 如果 collection 不存在，插入操作会自动创建 collection。

### 3.1.1. 插入单个文档

* 如果没有指定 `_id`， mongoDB 会自动添加。

```mongodb
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
```

### 3.1.2. 插入多个文档

```mongodb
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```


## 3.2. Query Documents

### 3.2.1. 查询 collection 中所有文档

```mongodb
db.inventory.find( {} )
```

### 3.2.2. 指定等式条件

```mongodb
db.inventory.find( { status: "D" } )
```

相当于 `SELECT * FROM inventory WHERE status = "D"`.


### 3.2.3. 使用查询操作符

```mongodb
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
```

[查看所有查询操作符](https://docs.mongodb.com/master/reference/operator/query/)


### 3.2.4. AND 多个条件

```
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
```

### 3.2.5. OR 多个条件

```
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```

相当于：`SELECT * FROM inventory WHERE status = "A" OR qty < 30`

### 3.2.6. 混合 AND 和 OR

```
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```

相当于：`SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")`


## 3.3. Update Documents

```
{
  <update operator>: { <field1>: <value1>, ... },
  <update operator>: { <field2>: <value2>, ... },
  ...
}
```

[update operator](https://docs.mongodb.com/manual/reference/operator/update/), 比如 `$set` 等。


### update

修改已存在的文档。可以修改特定字段，或者整个文档，取决于 [update parameter](https://docs.mongodb.com/manual/reference/method/db.collection.update/#update-parameter)



### 3.3.1. updateOne

模式：`db.collection.updateOne(<filter>, <update>, <options>)`

example1:

```
db.inventory.updateOne(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)
```

设置集合中 `item == paper` 的**第一个**文档，设置其 `size.uom, status` 字段，修改 `lastModified` 字段的值为当前时间，见 [$currentDate](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate).



### 3.3.2. updateMany

格式：`db.collection.updateMany(<filter>, <update>, <options>)`

example1:

```
db.inventory.updateMany(
   { "qty": { $lt: 50 } },
   {
     $set: { "size.uom": "in", status: "P" },
     $currentDate: { lastModified: true }
   }
)
```

更新 `qty < 50` 的**所有的**文档


### 3.3.3. replaceOne

格式：`db.collection.replaceOne(<filter>, <replacement>, <options>)`

替换文档只能包含键值对，不能包含 [update operators](https://docs.mongodb.com/manual/reference/operator/update) 表达式。

替换文档可以和原文档有不同的字段。在替换文档中，你可以省略 `_id` 字段，因为它是不可变的。如果你包含了 `_id` 字段，那么它和现在的值要一样。


example1:

```
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```

替换符合条件的**第一个**文档。


### 3.3.4. Upsert Option

如果 `updateOne(), updateMany(), or replaceOne()` 包含了 `upsert: true` 选项，当不存在匹配的文档时将插入一个新的文档，存在时则替换它。




# 4. 参考

* [www.mongodb.com](https://www.mongodb.com)