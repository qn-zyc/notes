<!-- TOC -->

- [1. install](#1-install)
    - [1.1. 在 mac os 上安装](#11-在-mac-os-上安装)
- [2. Run MongoDB](#2-run-mongodb)
- [3. Shell Methods](#3-shell-methods)
- [4. CRUD 操作](#4-crud-操作)
    - [4.1. Insert Documents](#41-insert-documents)
        - [4.1.1. 插入单个文档](#411-插入单个文档)
        - [4.1.2. 插入多个文档](#412-插入多个文档)
    - [4.2. Query Documents](#42-query-documents)
        - [4.2.1. 查询 collection 中所有文档](#421-查询-collection-中所有文档)
        - [4.2.2. 指定等式条件](#422-指定等式条件)
        - [4.2.3. 使用查询操作符](#423-使用查询操作符)
        - [4.2.4. AND 多个条件](#424-and-多个条件)
        - [4.2.5. OR 多个条件](#425-or-多个条件)
        - [4.2.6. 混合 AND 和 OR](#426-混合-and-和-or)
    - [4.3. Update Documents](#43-update-documents)
        - [4.3.1. update](#431-update)
        - [4.3.2. updateOne](#432-updateone)
        - [4.3.3. updateMany](#433-updatemany)
        - [4.3.4. replaceOne](#434-replaceone)
        - [4.3.5. Upsert Option](#435-upsert-option)
        - [4.3.6. inc 自增自减](#436-inc-自增自减)
    - [4.4. Delete Documents](#44-delete-documents)
        - [4.4.1. 删除所有文档](#441-删除所有文档)
        - [4.4.2. 删除符合条件的文档](#442-删除符合条件的文档)
- [5. 集合方法](#5-集合方法)
    - [5.1. aggregate](#51-aggregate)
        - [5.1.3. $group](#513-group)
        - [5.1.4. 示例](#514-示例)
            - [5.1.4.1. 统计一段时间内的总和](#5141-统计一段时间内的总和)
- [6. 参考](#6-参考)

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


# 3. Shell Methods

* [doc](https://docs.mongodb.com/manual/reference/method/)
* 帮助命令：`help`


# 4. CRUD 操作

## 4.1. Insert Documents

* 如果 collection 不存在，插入操作会自动创建 collection。

### 4.1.1. 插入单个文档

* 如果没有指定 `_id`， mongoDB 会自动添加。

```mongodb
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
```

### 4.1.2. 插入多个文档

```mongodb
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```


## 4.2. Query Documents

### 4.2.1. 查询 collection 中所有文档

```mongodb
db.inventory.find( {} )
```

### 4.2.2. 指定等式条件

```mongodb
db.inventory.find( { status: "D" } )
```

相当于 `SELECT * FROM inventory WHERE status = "D"`.


### 4.2.3. 使用查询操作符

```js
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
db.inventory.find( { date: { $lt: 1, $gt: 2} } )
```

[查看所有查询操作符](https://docs.mongodb.com/master/reference/operator/query/)


### 4.2.4. AND 多个条件

```
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
```

### 4.2.5. OR 多个条件

```
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```

相当于：`SELECT * FROM inventory WHERE status = "A" OR qty < 30`

### 4.2.6. 混合 AND 和 OR

```
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```

相当于：`SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")`





## 4.3. Update Documents

```
{
  <update operator>: { <field1>: <value1>, ... },
  <update operator>: { <field2>: <value2>, ... },
  ...
}
```

[update operator](https://docs.mongodb.com/manual/reference/operator/update/), 比如 `$set` 等。




### 4.3.1. update

修改已存在的文档。可以修改特定字段，或者整个文档，取决于 [update parameter](https://docs.mongodb.com/manual/reference/method/db.collection.update/#update-parameter)



### 4.3.2. updateOne

模式：`db.collection.updateOne(<filter>, <update>, <options>)`

example1:

```js
db.inventory.updateOne(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)
```

设置集合中 `item == paper` 的**第一个**文档，设置其 `size.uom, status` 字段，修改 `lastModified` 字段的值为当前时间，见 [$currentDate](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate).



### 4.3.3. updateMany

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


### 4.3.4. replaceOne

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


### 4.3.5. Upsert Option

如果 `updateOne(), updateMany(), or replaceOne()` 包含了 `upsert: true` 选项，当不存在匹配的文档时将插入一个新的文档，存在时则替换它。

示例:

```js
db.people.update(
   { name: "Andy" },
   {
      name: "Andy",
      rating: 1,
      score: 1
   },
   { upsert: true }
)
```


### 4.3.6. inc 自增自减

* `$inc` 可以接受正值和负值.
* 如果字段不存在会创建它并设置值为指定值.
* 在值为 null 的字段上使用 `$inc` 会产生一个错误.
* `$inc` 在单个文档中是原子性操作.
* [参考](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)

示例:

有如下文档:

```js
{
  _id: 1,
  sku: "abc123",
  quantity: 10,
  metrics: {
    orders: 2,
    ratings: 3.5
  }
}
```

现在想将 quantity 减 2, 将 metrics.orders 增 1:

```js
db.products.update(
   { sku: "abc123" },
   { $inc: { quantity: -2, "metrics.orders": 1 } }
)
```




## 4.4. Delete Documents

### 4.4.1. 删除所有文档

```
db.inventory.deleteMany({})
```

### 4.4.2. 删除符合条件的文档

```
{ <field1>: { <operator1>: <value1> }, ... }
```

[query operator](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors)


```
db.inventory.deleteMany({ status : "A" })
```


# 5. 集合方法

## 5.1. aggregate

做聚合和统计用的.

定义: `db.collection.aggregate(pipeline, options)`

* [文档](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/)
* [pipeline文档](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/)

### 5.1.3. $group

* `_id` 指定分组的关键字, 比如 `_id: {area: "$area", isp: "$isp"}`, 为 null 时则整个集合为一组.
* 字段经过运算后再分组: `_id: {date: {$divide: ["$date", 1000]}}`


### 5.1.4. 示例

#### 5.1.4.1. 统计一段时间内的总和

```js
db.col.aggregate(
    [
        {
            $match: {
                date: { $gt: 1, $lt: 9 }
            }
        },
        {
            $group: {
                _id: null,
                totalNum: { $sum: "$fieldName1" },
                count: { $sum: 1 }  // 计算个数
            }
        }
    ]
)
```




# 6. 参考

* [www.mongodb.com](https://www.mongodb.com)