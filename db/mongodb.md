<!-- TOC -->

- [install](#install)
    - [在 mac os 上安装](#在-mac-os-上安装)
    - [在linux上安装](#在linux上安装)
- [Run MongoDB](#run-mongodb)
- [Shell Methods](#shell-methods)
    - [系统](#系统)
    - [db](#db)
    - [runCommand](#runcommand)
        - [finalize](#finalize)
        - [keyf](#keyf)
    - [findAndModify](#findandmodify)
- [CRUD 操作](#crud-操作)
    - [Insert Documents](#insert-documents)
        - [插入单个文档](#插入单个文档)
        - [插入多个文档](#插入多个文档)
        - [插入时间](#插入时间)
    - [Query Documents](#query-documents)
        - [查询 collection 中所有文档](#查询-collection-中所有文档)
        - [指定等式条件](#指定等式条件)
        - [使用查询操作符](#使用查询操作符)
        - [AND 多个条件](#and-多个条件)
        - [OR 多个条件](#or-多个条件)
        - [混合 AND 和 OR](#混合-and-和-or)
    - [Update Documents](#update-documents)
        - [update](#update)
        - [updateOne](#updateone)
        - [updateMany](#updatemany)
        - [replaceOne](#replaceone)
        - [Upsert Option](#upsert-option)
        - [inc 自增自减](#inc-自增自减)
        - [更新集合](#更新集合)
    - [Delete Documents](#delete-documents)
        - [删除所有文档](#删除所有文档)
        - [删除符合条件的文档](#删除符合条件的文档)
        - [remove删除](#remove删除)
- [集合方法](#集合方法)
    - [删除集合](#删除集合)
    - [aggregate](#aggregate)
        - [$group](#group)
        - [示例](#示例)
            - [统计一段时间内的总和](#统计一段时间内的总和)
- [索引](#索引)
    - [创建索引](#创建索引)
        - [复合索引](#复合索引)
    - [查看索引](#查看索引)
    - [删除索引](#删除索引)
    - [唯一索引](#唯一索引)
    - [system.indexes](#systemindexes)
    - [后台创建索引](#后台创建索引)
    - [过期索引](#过期索引)
- [备份与恢复](#备份与恢复)
- [性能调优](#性能调优)
    - [explain](#explain)
- [引擎](#引擎)
    - [WiredTiger](#wiredtiger)
- [复制集](#复制集)
    - [操作secondary](#操作secondary)
- [参考](#参考)

<!-- /TOC -->


# install

* [下载](https://www.mongodb.com/download-center?jmp=nav#community)
* [安装文档](https://docs.mongodb.com/manual/installation/)

## 在 mac os 上安装

1. 下载二进制包: `curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.5.tgz`
2. 解压: `tar -zxvf mongodb-osx-x86_64-3.4.5.tgz`
3. 配置环境变量 PATH，路径是解压后的目录中的 bin 目录：`export PATH=$PATH:/mongoDB/mongodb-osx-x86_64-3.4.5/bin`


## 在linux上安装

1. 下载 `curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1404-3.2.0.tgz`
2. 解压 `tar -zxvf mongodb-linux-x86_64-ubuntu1404-3.2.0.tgz`
3. 移动 `mv  mongodb-linux-x86_64-ubuntu1404-3.2.0/ /usr/local/mongodb`
4. 添加环境变量 `export PATH=/usr/local/mongodb/bin:$PATH`


# Run MongoDB

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



# Shell Methods

* [doc](https://docs.mongodb.com/manual/reference/method/)
* 帮助命令：`help`

## 系统
* 查看系统使用的引擎: `db.serverStatus().storageEngine`
* 查看系统状态: `db.serverStatus()`


## db

- 创建数据库: `use DATABASE_NAME;`
- 删除数据库: `db.dropDatabase()`, 删除前使用 `use` 选中一个数据库.



## runCommand

统计最近时间的ips值:

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


累加:

```js
db.runCommand(
    {
        "group": {
            "ns": "bili",
            "key": {"ts": true, "view.province": true, "view.isp": true},
            "initial": {"total": 0, "size": 0, "elapse": 0, "is_slow": 0},
            "$reduce": function(doc, prev) {
                prev.total += 1;
                prev.size += doc.size;
                prev.elapse += doc.elapse;
                prev.is_slow += doc.is_slow;
            }
        }
    }
)
```


### finalize

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


### keyf

当使用key时是区分大小写的, 如果想将文档中的key做一些处理可以使用 `$keyf`;

```js
"$keyf" : function(doc){return {"name" : doc.name.toLowerCase()};}
```

上面是将 name 转换成小写之后再作为分组条件, 注意 `keyf` 最后也需要返回一个文档.


## findAndModify
* [文档](https://docs.mongodb.com/manual/reference/command/findAndModify/)
* 只会修改并返回一个文档.

模拟队列, 提取出 unix 最小的一条记录, 并锁定:
```js
db.runCommand(
    {
        findAndModify: "queue",
        query: { lock: 0 },
        sort: { unix: 1 },
        update: { $set: { lock: 1 } },
    }
)
```



# CRUD 操作

## Insert Documents

* 如果 collection 不存在，插入操作会自动创建 collection。

### 插入单个文档

* 如果没有指定 `_id`， mongoDB 会自动添加。

```mongodb
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
```

### 插入多个文档

```mongodb
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```


### 插入时间

```js
db.col.insert({mark:1, mark_time:new Date()})
db.col.insert({mark:2, mark_time:Date()})
```

`new Date()` 的结果是 `ISODate("2013-02-22T03:03:37.312Z")`, 以 UTC 存储.

`Date()` 存的是字符串 `Fri Feb 22 2013 11:03:40 GMT+0800`.


## Query Documents

### 查询 collection 中所有文档

```mongodb
db.inventory.find( {} )
```

### 指定等式条件

```mongodb
db.inventory.find( { status: "D" } )
```

相当于 `SELECT * FROM inventory WHERE status = "D"`.


### 使用查询操作符

```js
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
db.inventory.find( { date: { $lt: 1, $gt: 2} } )
```

[查看所有查询操作符](https://docs.mongodb.com/master/reference/operator/query/)


### AND 多个条件

```
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
```

### OR 多个条件

```
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```

相当于：`SELECT * FROM inventory WHERE status = "A" OR qty < 30`

### 混合 AND 和 OR

```
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```

相当于：`SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")`





## Update Documents

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

* 如果update中只包含键值对时, 将更新所有字段(会删除没有更新的字段), 如果包含了更新操作符(比如`$set`)则只更新指定的字段.



### updateOne

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



### updateMany

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


### replaceOne

格式：`db.collection.replaceOne(<filter>, <replacement>, <options>)`

替换文档只能包含键值对，不能包含 [update operators](https://docs.mongodb.com/manual/reference/operator/update) 表达式。

替换文档可以和原文档有不同的字段。在替换文档中，你可以省略 `_id` 字段，因为它是不可变的。如果你包含了 `_id` 字段，那么它和现在的值要一样。


example1:

```js
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```

替换符合条件的**第一个**文档。


### Upsert Option

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


### inc 自增自减

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


### 更新集合

不存在时添加:

```js
db.queue.updateMany(
    {lock: 1},
    {$addToSet: {set_name: "hello"}}
)
db.queue.updateMany(
    {lock: 1},
    {$addToSet: {set_name: {$each: ["1", "2"]}}}
)
```

* `$addToSet` 不会添加已经存在的元素.
* 如果传递的是一个数组: `{$addToSet: {set_name: ["a", "b"]}}`, 那么数组被当做一个元素放入.
* 如果想将数组中每个元素分开插入, 使用 `$each`: `{$addToSet: {set_name: {$each: ["a", "b"]}}}`



## Delete Documents

### 删除所有文档

```
db.inventory.deleteMany({})
```

### 删除符合条件的文档

```
{ <field1>: { <operator1>: <value1> }, ... }
```

[query operator](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors)


```
db.inventory.deleteMany({ status : "A" })
```


### remove删除

有些版本不支持 delete, 可以使用 `remove()`, 用法应该差不多(我猜的)



# 集合方法

## 删除集合

```js
db.colName.drop()
```

## aggregate

做聚合和统计用的.

定义: `db.collection.aggregate(pipeline, options)`

* [文档](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/)
* [pipeline文档](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/)

### $group

* `_id` 指定分组的关键字, 比如 `_id: {area: "$area", isp: "$isp"}`, 为 null 时则整个集合为一组.
* 字段经过运算后再分组: `_id: {date: {$divide: ["$date", 1000]}}`


### 示例

#### 统计一段时间内的总和

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





# 索引

## 创建索引

```js
db.test.ensureIndex({"username":1})
```

* 1表示升序, -1表示降序.
* 为内嵌文档创建索引: `db.test.ensureIndex({"comments.date":1})`
* 为索引指定名称: `db.test.ensureIndex({"username":1},{"name":"testindex"}) `

### 复合索引

```js
db.test.ensureIndex({"username":1, "age":-1})
```

如果想使用索引, 查询条件中必须包含符合索引中前N个序列, 顺序可以不和创建时一样.

## 查看索引

```js
db.test.getIndexes()
```

## 删除索引

```js
db.test.dropIndex({"username":1})
```


## 唯一索引

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


## system.indexes

`system.indexes` 包含了每个索引的详细信息, 可以使用下面的语句查询:

```js
db.system.indexes.find()
```

## 后台创建索引

```js
db.test.ensureIndex({"username":1},{"background":true})
```

* 后台创建索引时不会阻塞其他操作.
* 阻塞方式创建索引时效率更高.



## 过期索引

```js
db.colName.createIndex( { "createTime": 1 }, { expireAfterSeconds: 60*60 } )
```

* `createTime` 是要创建索引的字段名称, `expireAfterSeconds` 是过期时间, 单位秒.
* The `_id` field does not support TTL indexes.
* 你不能在一个已经有索引的字段上创建TTL索引。
* 如果文档不存在被索引的字段，该文档不会过期。
* 如果被索引的字段不是日期 BSON type 或是日期数组 BSON types，文档不会过期。
* TTL索引不能是复合索引（不能有多个字段）。
* 如果TTL字段容纳了一个数组并且在索引中有多个日期类型的数据，文档将在 最小的 （即最早的）日期等于过期临界值时过期。
* 你不能在一个固定集合上创建TTL索引，因为MongoDB不能从固定集合中删除文档。
* 你不能使用 ensureIndex() 改变 expireAfterSeconds 的值。作为替代，请使用 collMod 数据库命令连同 index 集合标志。
* 当你在 background 创建一个TTL索引时，TTL线程可以在创建索引的同时开始删除文档。如果你在前台创建一个TTL索引，一旦索引完成创建，MongoDB就开始删除过期的文档。
* 删除过期文档的后台任务每60秒运行一次。因此，文档在他们过期后但在后台任务运行并完成前仍会留在集合中。
* 删除操作持续的时间由你的 mongod 实例的工作负荷决定。因此，某些时候，过期的数据可能会在两次后台任务运行之间存在 超过 60秒的时间。




# 备份与恢复

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



# 性能调优

## explain

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

在 aggregate 中调优:

```js
db.bili.aggregate(
    [
        {
            $match: {
                ts: { $gte: 0, $lt: 35079478}
            }
        },
        {
            $group: {
                _id: {"ts": "$ts", "prov": "$prov", "isp": "$isp", "domain": "$domain", "hit_miss": "$hit_miss"},
                total: { $sum: 1 },
                size: { $sum: "$size" },
                elapse: { $sum: "$elapse" },
                is_slow: { $sum: "$is_slow" }
            }
        }
    ],
    {
        explain: true
    }
)

{
	"stages" : [
		{
			"$cursor" : {
				"query" : {
					"ts" : {
						"$gte" : 0,
						"$lt" : 35079478
					}
				},
				"fields" : {
					"domain" : 1,
					"elapse" : 1,
					"hit_miss" : 1,
					"is_slow" : 1,
					"isp" : 1,
					"prov" : 1,
					"size" : 1,
					"ts" : 1,
					"_id" : 0
				},
				"queryPlanner" : {
					"plannerVersion" : 1,
					"namespace" : "kuaishoulog.bili",
					"indexFilterSet" : false,
					"parsedQuery" : {
						"$and" : [
							{
								"ts" : {
									"$lt" : 35079478
								}
							},
							{
								"ts" : {
									"$gte" : 0
								}
							}
						]
					},
					"winningPlan" : {
						"stage" : "FETCH",
						"inputStage" : {
							"stage" : "IXSCAN",
							"keyPattern" : {
								"ts" : 1,
								"prov" : 1,
								"isp" : 1,
								"domain" : 1,
								"hit_miss" : 1
							},
							"indexName" : "ts_1_prov_1_isp_1_domain_1_hit_miss_1",
							"isMultiKey" : false,
							"multiKeyPaths" : {
								"ts" : [ ],
								"prov" : [ ],
								"isp" : [ ],
								"domain" : [ ],
								"hit_miss" : [ ]
							},
							"isUnique" : false,
							"isSparse" : false,
							"isPartial" : false,
							"indexVersion" : 2,
							"direction" : "forward",
							"indexBounds" : {
								"ts" : [
									"[0.0, 35079478.0)"
								],
								"prov" : [
									"[MinKey, MaxKey]"
								],
								"isp" : [
									"[MinKey, MaxKey]"
								],
								"domain" : [
									"[MinKey, MaxKey]"
								],
								"hit_miss" : [
									"[MinKey, MaxKey]"
								]
							}
						}
					},
					"rejectedPlans" : [ ]
				}
			}
		},
		{
			"$group" : {
				"_id" : {
					"ts" : "$ts",
					"prov" : "$prov",
					"isp" : "$isp",
					"domain" : "$domain",
					"hit_miss" : "$hit_miss"
				},
				"total" : {
					"$sum" : {
						"$const" : 1
					}
				},
				"size" : {
					"$sum" : "$size"
				},
				"elapse" : {
					"$sum" : "$elapse"
				},
				"is_slow" : {
					"$sum" : "$is_slow"
				}
			}
		}
	],
	"ok" : 1
}
```

* stage的含义
    * COLLSCAN 全表扫描
    * IXSCAN 索引扫描
    * FETCH 根据索引去检索指定document
    * SHARD_MERGE 将各个分片返回数据进行merge
    * SORT 在内存中进行了排序（与老版本的scanAndOrder:true一致）
    * LIMIT 使用limit限制返回数
    * SKIP 使用skip进行跳过
    * IDHACK 针对_id进行查询
    * SHARDING_FILTER 通过mongos对分片数据进行查询
    * COUNT 利用db.coll.explain().count()之类进行count运算
    * COUNTSCAN count不使用用Index进行count时的stage返回
    * COUNT_SCAN count使用了Index进行count时的stage返回
    * SUBPLA 未使用到索引的$or查询的stage返回
    * TEXT 使用全文索引进行查询时候的stage返回
    * PROJECTION 限定返回字段时候stage的返回


# 引擎
* 从3.0开始引入可插拔引擎, 可选MMAPV1、WiredTiger、MongoRocks、TokuSE等.


## WiredTiger

WiredTiger的优势:
* 性能及并发更好：wiredTiger更好的利用了现代的多核架构，因此性能上会更好。此外，wiredTiger采用了document-based 锁机制，锁粒度更小，从而支持更高的并发。
* wiredTiger支持数据的加密和压缩
* 索引的存储采用了前缀压缩，从而索引更小，降低了内存的占用

Cache: cache的大小极大的影响了wireTiger存储引擎的性能，默认的，mongo将cache的大小设置为内存的一半.


# 复制集
* 复制集的成员最多50个, 最多7个具有投票权, [文档](https://docs.mongodb.com/manual/core/replica-set-architectures/)
* [复制集和主从模式的对比](http://blog.csdn.net/canot/article/details/50739359)

## 操作secondary

默认情况下，Secondary是不提供服务的，即不能读和写。会提示：`error: { "$err" : "not master and slaveOk=false", "code" : 13435 }`

在特殊情况下需要读的话则需要：`rs.slaveOk()` ，只对当前连接有效。


# 参考

* [www.mongodb.com](https://www.mongodb.com)
* [MongoDB学习笔记(索引)](http://www.cnblogs.com/stephen-liu74/archive/2012/08/01/2561557.html)
* [mongo wiredTiger存储引擎相关](http://blog.csdn.net/weiyuanke/article/details/72724052)
* [学习MongoDB–聚合（初级聚合函数使用）](http://blog.sae.sina.com.cn/archives/1495)
* [MongoDB 副本集的原理、搭建、应用](http://www.cnblogs.com/zhoujinyi/p/3554010.html)
* [官网中关于复制集](https://docs.mongodb.com/manual/replication/)
