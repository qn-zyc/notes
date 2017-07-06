<!-- TOC -->

- [1. 概述](#1-概述)
    - [1.1. 概念](#11-概念)
- [2. 命令](#2-命令)
- [3. 数据库](#3-数据库)
- [4. Writing and exploring data](#4-writing-and-exploring-data)

<!-- /TOC -->

# 1. 概述

* [官网](https://www.influxdata.com/)
* [doc](https://docs.influxdata.com/influxdb/v1.2/)
* [reference](https://docs.influxdata.com/influxdb/v1.2/query_language/spec/)

## 1.1. 概念

* measurement like table, 主键总是时间, tags 和 fields 是列，tags 被索引了，fields 没有被索引。
* measurement 至少一个键值对，比如 `value=0.6`。
* null 值不被存储。


# 2. 命令

* 启动服务 `influxd`
* 启动命令行 `influx`


# 3. 数据库

* `CREATE DATABASE mydb`: 创建数据库，如果数据库名包含除了ASCII字符，数字，下划线还包括其他字符，需要使用双引号。
* `SHOW DATABASES`: 显示已存在的数据库。`_internal` 是 influxDB 存储内部运行数据的。
* `USE mydb`: 每个查询都需要制定数据库，使用这个语句避免每次都指定。

# 4. Writing and exploring data

Points are written to InfluxDB using the Line Protocol, which follows the following format:

```
<measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
```

examples:

```
cpu,host=serverA,region=us_west value=0.64
payment,device=mobile,product=Notepad,method=credit billed=33,licenses=3i 1434067467100293230
stock,symbol=AAPL bid=127.46,ask=127.48
temperature,machine=unit42,type=assembly external=25,internal=37 1434067467000000000
```

[更多关于Line Protocol](https://docs.influxdata.com/influxdb/v1.2/write_protocols/line_protocol_reference/)

写入数据：

```
> INSERT cpu,host=serverA,region=us_west value=0.64
```

查询数据：

```
> SELECT "host", "region", "value" FROM "cpu"
name: cpu
---------
time		    	                     host     	region   value
2015-10-21T19:28:07.580664347Z  serverA	  us_west	 0.64
```

```
> SELECT * FROM "temperature"
--
> SELECT * FROM /.*/ LIMIT 1
--
> SELECT * FROM "cpu_load_short"
--
> SELECT * FROM "cpu_load_short" WHERE "value" > 0.9
```

