<!-- TOC -->

- [1. install](#1-install)
    - [1.1. 在 mac os 上安装](#11-在-mac-os-上安装)
    - [1.2. 在linux上安装](#12-在linux上安装)
- [2. Run MongoDB](#2-run-mongodb)
- [3. Shell Methods](#3-shell-methods)
    - [3.1. 系统](#31-系统)
    - [3.2. db](#32-db)
    - [3.3. runCommand](#33-runcommand)
        - [3.3.1. finalize](#331-finalize)
        - [3.3.2. keyf](#332-keyf)
- [4. CRUD 操作](#4-crud-操作)
    - [4.1. Insert Documents](#41-insert-documents)
        - [4.1.3. 插入单个文档](#413-插入单个文档)
        - [4.1.4. 插入多个文档](#414-插入多个文档)
        - [4.1.5. 插入时间](#415-插入时间)
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
        - [4.4.3. remove删除](#443-remove删除)
- [5. 集合方法](#5-集合方法)
    - [5.1. 删除集合](#51-删除集合)
    - [5.2. aggregate](#52-aggregate)
        - [5.2.1. $group](#521-group)
        - [5.2.2. 示例](#522-示例)
            - [5.2.2.1. 统计一段时间内的总和](#5221-统计一段时间内的总和)
- [6. 索引](#6-索引)
    - [6.1. 创建索引](#61-创建索引)
        - [6.1.3. 复合索引](#613-复合索引)
    - [6.2. 查看索引](#62-查看索引)
    - [6.3. 删除索引](#63-删除索引)
    - [6.4. 唯一索引](#64-唯一索引)
    - [6.5. system.indexes](#65-systemindexes)
    - [6.6. 后台创建索引](#66-后台创建索引)
    - [6.7. 过期索引](#67-过期索引)
- [7. 备份与恢复](#7-备份与恢复)
- [8. 性能调优](#8-性能调优)
    - [8.1. explain](#81-explain)
- [9. 引擎](#9-引擎)
    - [9.1. WiredTiger](#91-wiredtiger)
- [10. 参考](#10-参考)

<!-- /TOC -->


# 1. install

* [下载](https://www.mongodb.com/download-center?jmp=nav#community)
* [安装文档](https://docs.mongodb.com/manual/installation/)

## 1.1. 在 mac os 上安装

1. 下载二进制包: `curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.5.tgz`
2. 解压: `tar -zxvf mongodb-osx-x86_64-3.4.5.tgz`
3. 配置环境变量 PATH，路径是解压后的目录中的 bin 目录：`export PATH=$PATH:/mongoDB/mongodb-osx-x86_64-3.4.5/bin`


## 1.2. 在linux上安装

1. 下载 `curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1404-3.2.0.tgz`
2. 解压 `tar -zxvf mongodb-linux-x86_64-ubuntu1404-3.2.0.tgz`
3. 移动 `mv  mongodb-linux-x86_64-ubuntu1404-3.2.0/ /usr/local/mongodb`
4. 添加环境变量 `export PATH=/usr/local/mongodb/bin:$PATH`


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

* 查看帮助: `mongod -h`
* 查看版本: `mongod --version`



# 3. Shell Methods

* [doc](https://docs.mongodb.com/manual/reference/method/)
* 帮助命令：`help`

## 3.1. 系统
* 查看系统使用的引擎: `db.serverStatus().storageEngine`
* 查看系统状态: `db.serverStatus()`


## 3.2. db
* 删除数据库: `db.dropDatabase()`, 删除前使用 `use` 选中一个数据库.



## 3.3. runCommand

```js
db.runCommand(
    {
        "group": {
            "ns": "task_result",
            "key": {"domain": true, "country": true, "isp": true},
            "initial": {"time": 0},
            "$reduce": function(doc, prev) {
                if (doc.create_at > prev.time) {
                    prev.time = doc.create_at;
                    prev.ips = doc.ips;
                }
            },
            "condition":{"create_at":{"$lte": new Date("2017-08-03T11:48:00")}}
        }
    }
)
```

* `ns`: 指定要操作的集合.
* `key`: 是分组使用的键.
* `initial`: 初始化的文档, 每个组都会有一个初始化的文档, 最后的结果中也是这个文档加上分组的key.
* `$reduce`: 聚合函数, doc是当前文档, prev是累加器的文档, 初始时的文档.
* `condition`: 过滤文档的条件, 也可以缩写成 `cond` 或 `q`.

### 3.3.1. finalize

```js
> db.runCommand({"group" : {
... "ns" : "blog",
... "key" : {"author" : true},
... "initial" : {"tag" : {}},
... "$reduce" : function(doc, prev) {
...                 for(var i in doc.tag){
...                     if(doc.tag[i] in prev.tag){
...                         prev.tag[doc.tag[i]]++;
...                     } else {
...                         prev.tag[doc.tag[i]] = 1;
...                     }
...                 }
...             },
... "finalize" : function(reducedoc) {
...                  var mostpopular = 0;
...                  for(var i in reducedoc.tag) {
...                      if(reducedoc.tag[i] > mostpopular) {
...                          mostpopular = reducedoc.tag[i];
...                          reducedoc.usemosttag = i;
...                      }
...                  }
...                  delete reducedoc.tag;
...              }
... }});
{
        "retval" : [
                {
                        "author" : "jim",
                        "usemosttag" : "cat"
                },
                {
                        "author" : "tom",
                        "usemosttag" : "db"
                }
        ],
        "count" : 5,
        "keys" : 2,
        "ok" : 1
}
```

这是统计每个作者写博客时最常使用的tag, 先是使用 reduce 统计每个作者使用的每个tag的次数, 然后使用 finalize 对每个聚合文档进一步处理得出最常使用的tag.


### 3.3.2. keyf

当使用key时是区分大小写的, 如果想将文档中的key做一些处理可以使用 `$keyf`;

```js
"$keyf" : function(doc){return {"name" : doc.name.toLowerCase()};}
```

上面是将 name 转换成小写之后再作为分组条件, 注意 `keyf` 最后也需要返回一个文档.





# 4. CRUD 操作

## 4.1. Insert Documents

* 如果 collection 不存在，插入操作会自动创建 collection。

### 4.1.3. 插入单个文档

* 如果没有指定 `_id`， mongoDB 会自动添加。

```mongodb
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
```

### 4.1.4. 插入多个文档

```mongodb
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```


### 4.1.5. 插入时间

```js
db.col.insert({mark:1, mark_time:new Date()})
db.col.insert({mark:2, mark_time:Date()})
```

`new Date()` 的结果是 `ISODate("2013-02-22T03:03:37.312Z")`, 以 UTC 存储.

`Date()` 存的是字符串 `Fri Feb 22 2013 11:03:40 GMT+0800`.


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


### 4.4.3. remove删除

有些版本不支持 delete, 可以使用 `remove()`, 用法应该差不多(我猜的)



# 5. 集合方法

## 5.1. 删除集合

```js
db.colName.drop()
```

## 5.2. aggregate

做聚合和统计用的.

定义: `db.collection.aggregate(pipeline, options)`

* [文档](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/)
* [pipeline文档](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/)

### 5.2.1. $group

* `_id` 指定分组的关键字, 比如 `_id: {area: "$area", isp: "$isp"}`, 为 null 时则整个集合为一组.
* 字段经过运算后再分组: `_id: {date: {$divide: ["$date", 1000]}}`


### 5.2.2. 示例

#### 5.2.2.1. 统计一段时间内的总和

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





# 6. 索引

## 6.1. 创建索引

```js
db.test.ensureIndex({"username":1})
```

* 1表示升序, -1表示降序.
* 为内嵌文档创建索引: `db.test.ensureIndex({"comments.date":1})`
* 为索引指定名称: `db.test.ensureIndex({"username":1},{"name":"testindex"}) `

### 6.1.3. 复合索引

```js
db.test.ensureIndex({"username":1, "age":-1})
```

如果想使用索引, 查询条件中必须包含符合索引中前N个序列, 顺序可以不和创建时一样.

## 6.2. 查看索引

```js
db.test.getIndexes()
```

## 6.3. 删除索引

```js
db.test.dropIndex({"username":1})
```


## 6.4. 唯一索引

```js
db.test.ensureIndex({"userid":1},{"unique":true})
```

* 插入相同 userid 的文档时 mongodb 将会报错.
* 如果插入的文档中不包含 userid, 缺省为 null, 相同 null 值也会报错.

在创建唯一索引时将重复的记录删除, 只保留第一个文档:

```js
db.test.ensureIndex({"userid":1},{"unique":true,"dropDups":true}) 
```

创建唯一复合索引:

```js
db.test.ensureIndex({"userid":1,"age":1},{"unique":true})
```


## 6.5. system.indexes

`system.indexes` 包含了每个索引的详细信息, 可以使用下面的语句查询:

```js
db.system.indexes.find()
```

## 6.6. 后台创建索引

```js
db.test.ensureIndex({"username":1},{"background":true})
```

* 后台创建索引时不会阻塞其他操作.
* 阻塞方式创建索引时效率更高.



## 6.7. 过期索引

```js
db.colName.createIndex( { "createTime": 1 }, { expireAfterSeconds: 60*60 } )
```

* `createTime` 是要创建索引的字段名称, `expireAfterSeconds` 是过期时间, 单位秒.



# 7. 备份与恢复

```shell
/usr/local/mongodb3.4.6/bin/mongodump -h=127.0.0.1:27017 -d kuaishoulog -o mongo_backup
```

* `-d` 指定db.
* `-o` 指定备份到哪个目录.


```shell
/usr/local/mongodb3.4.6/bin/mongorestore -h=127.0.0.1:27018 -d kuaishoulog --drop mongo_backup/kuaishoulog/
```

* `-d` 指定要恢复的数据库.
* `--drop` 表示在恢复前删除集合（若存在）。否则，数据就会与现有集合数据合并，可能会覆盖一些文档.



# 8. 性能调优

## 8.1. explain

```js
> db.test.find().explain()
{
    "cursor" : "BasicCursor",
    "nscanned" : 1,
    "nscannedObjects" : 1,
    "n" : 1,
    "millis" : 0,
    "nYields" : 0,
    "nChunkSkips" : 0,
    "isMultiKey" : false,
    "indexOnly" : false,
    "indexBounds" : {

    }    
}
```

* `cursor: BasicCursor` 表示没有使用索引.
* `nscanned` 表示查询了多少个文档.
* `n` 表示返回的文档数量.
* `millis` 表示耗时.


# 9. 引擎
* 从3.0开始引入可插拔引擎, 可选MMAPV1、WiredTiger、MongoRocks、TokuSE等.


## 9.1. WiredTiger

WiredTiger的优势:
* 性能及并发更好：wiredTiger更好的利用了现代的多核架构，因此性能上会更好。此外，wiredTiger采用了document-based 锁机制，锁粒度更小，从而支持更高的并发。
* wiredTiger支持数据的加密和压缩
* 索引的存储采用了前缀压缩，从而索引更小，降低了内存的占用

Cache: cache的大小极大的影响了wireTiger存储引擎的性能，默认的，mongo将cache的大小设置为内存的一半.



# 10. 参考

* [www.mongodb.com](https://www.mongodb.com)
* [MongoDB学习笔记(索引)](http://www.cnblogs.com/stephen-liu74/archive/2012/08/01/2561557.html)
* [mongo wiredTiger存储引擎相关](http://blog.csdn.net/weiyuanke/article/details/72724052)
* [学习MongoDB–聚合（初级聚合函数使用）](http://blog.sae.sina.com.cn/archives/1495)