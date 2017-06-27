<!-- TOC -->

- [1. install](#1-install)
    - [1.1. 在 mac os 上安装](#11-在-mac-os-上安装)
- [2. Run MongoDB](#2-run-mongodb)
- [3. CRUD 操作](#3-crud-操作)
    - [3.1. Insert Documents](#31-insert-documents)
        - [3.1.1. 插入单个文档](#311-插入单个文档)
        - [3.1.2. 插入多个文档](#312-插入多个文档)
    - [Query Documents](#query-documents)
        - [查询 collection 中所有文档](#查询-collection-中所有文档)
        - [指定等式条件](#指定等式条件)
        - [使用查询操作符](#使用查询操作符)
        - [AND 多个条件](#and-多个条件)
        - [OR 多个条件](#or-多个条件)
        - [混合 AND 和 OR](#混合-and-和-or)
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

```mongodb
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
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





# 4. 参考

* [www.mongodb.com](https://www.mongodb.com)