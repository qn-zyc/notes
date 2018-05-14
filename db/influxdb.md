<!-- TOC -->

- [概述](#概述)
    - [概念](#概念)
    - [网络](#网络)
- [安装](#安装)
- [命令](#命令)
    - [保留策略 retention policy](#保留策略-retention-policy)
- [配置](#配置)
    - [使用自己的配置启动](#使用自己的配置启动)
- [数据库](#数据库)
- [Writing and exploring data](#writing-and-exploring-data)
- [HTTP API](#http-api)
    - [数据库](#数据库-1)
    - [写数据](#写数据)
    - [查询](#查询)
- [influxQL](#influxql)
    - [时间](#时间)
    - [select](#select)
    - [where](#where)
    - [group](#group)
        - [group by time intervals](#group-by-time-intervals)
    - [order](#order)
        - [order by time desc](#order-by-time-desc)
    - [limit](#limit)

<!-- /TOC -->

# 概述

* [官网](https://www.influxdata.com/)
* [doc](https://docs.influxdata.com/influxdb/v1.2/)
* [reference](https://docs.influxdata.com/influxdb/v1.2/query_language/spec/)

## 概念

* measurement like table, 主键总是时间, tags 和 fields 是列，tags 被索引了，fields 没有被索引。
* measurement 至少一个键值对，比如 `value=0.6`。
* null 值不被存储。
* NTP(Network Time Protocol) 同步主机间的时间, 数据中的时间戳是本地时间的 UTC 时间.
* retention policy: 数据保留策略.
* series是共享同一个retention policy，measurement以及tag set的数据集合.
* point则是同一个series中具有相同时间的field set，points相当于SQL中的数据行.
* database: influxdb不是一个完整的CRUD数据库，它更像是一个CR-ud数据库。它优先考虑的是增加和读取数据而不是更新和删除数据的性能，而且它阻止了某些更新和删除行为使得创建和读取数据更加高效.

## 网络
* `8086` 端口用于客户端和服务端通信, HTTP 接口.
* `8088` 端口用于 RPC 服务, restore and backup.
* 一些插件可能需要其它端口, 这些都可以在配置文件中配置.


# 安装

mac:

```bash
brew update
brew install influxdb
```


# 命令

* 启动服务 `influxd`
* 启动命令行 `influx`
* 时间戳格式化: `influx -precision rfc3339`

## 保留策略 retention policy
创建保留策略: `CREATE RETENTION POLICY "two_hours" ON "food_data" DURATION 2h REPLICATION 1 DEFAULT`
* `two_hours` 是策略名, `food_data` 是数据库名.
* `DURATION 2h` 数据保留2个小时.
* `REPLICATION 1` 必须的参数, 单节点总是设置为1.
* `DEFAULT` 设置为默认保留策略.
* 当创建数据库 `food_data` 时, influxDB 会创建一个名为 `autogen` 的默认保留策略, 这个策略 has an infinite retention period.

create a non-DEFAULT RP: `CREATE RETENTION POLICY "a_year" ON "food_data" DURATION 52w REPLICATION 1`
* `52w` 52周.


# 配置

## 使用自己的配置启动
1. 生成配置模板: `influxd config > my_influxdb.conf`
1. `-config` 参数 `influxd -config /etc/influxdb/influxdb.conf`
2. 在环境变量 `INFLUXDB_CONFIG_PATH` 中指定.

先检查 `-config`, 然后是环境变量.


# 数据库

* `CREATE DATABASE mydb`: 创建数据库，如果数据库名包含除了ASCII字符，数字，下划线还包括其他字符，需要使用双引号。
* `SHOW DATABASES`: 显示已存在的数据库。`_internal` 是 influxDB 存储内部运行数据的。
* `USE mydb`: 每个查询都需要制定数据库，使用这个语句避免每次都指定。
* `DROP DATABASE mydb`: 删除数据库。
* `show measurements`: 显示该数据库中所有的表。
* `drop measurement "measurement_name"`: 删除表。



# Writing and exploring data

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

# HTTP API
* [write API](https://docs.influxdata.com/influxdb/v1.3/tools/api/#write)

## 数据库

```shell
curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"
```

## 写数据

```shell
curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000'
```

多行(使用换行符分隔):

```shell
curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server02 value=0.67
cpu_load_short,host=server02,region=us-west value=0.55 1422568543702900257
cpu_load_short,direction=in,host=server01,region=us-west value=2.0 1422568543702900257'
```

使用文件写数据:
* 超过5000个点时最好分批上传, 如果文件太大, http 超时, 但是 influxDB 还在处理, 那么你就得不到响应信息.

```shell
> cat cpu_data.txt
cpu_load_short,host=server02 value=0.67
cpu_load_short,host=server02,region=us-west value=0.55 1422568543702900257
cpu_load_short,direction=in,host=server01,region=us-west value=2.0 1422568543702900257

> curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary @cpu_data.txt
```

## 查询

```shell
curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=SELECT \"value\" FROM \"cpu_load_short\" WHERE \"region\"='us-west'"
```

多个查询:

```shell
curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=SELECT \"value\" FROM \"cpu_load_short\" WHERE \"region\"='us-west';SELECT count(\"value\") FROM \"cpu_load_short\" WHERE \"region\"='us-west'"
```


# influxQL

## 时间

过滤时间: `SELECT * FROM "foodships" WHERE "planet" = 'Saturn' AND time > '2015-04-16 12:00:01'`
* 时间格式: `YYYY-MM-DD HH:MM:SS.mmm`, mmm 是毫秒, 还可以指定微秒和纳秒.

相对当前时间: `SELECT * FROM "foodships" WHERE time > now() - 1h`

时间间隔:

| Letter |   Meaning    |
| ------ | ------------ |
| ns     | nanoseconds  |
| u or µ | microseconds |
| ms     | milliseconds |
| s      | seconds      |
| m      | minutes      |
| h      | hours        |
| d      | days         |
| w      | weeks        |

## select
Syntax: `SELECT <field_key>[,<field_key>,<tag_key>] FROM <measurement_name>[,<measurement_name>]`

指定 field | tag:
* `SELECT "level description"::field,"location"::tag,"water_level"::field FROM "h2o_feet"`
* 所有的field: `SELECT *::field FROM "h2o_feet"`

执行运算: `SELECT ("water_level" * 2) + 4 from "h2o_feet"`

## where
Syntax: `SELECT_clause FROM_clause WHERE <conditional_expression> [(AND|OR) <conditional_expression> [...]]`

conditional_expression: `field_key <operator> ['string' | boolean | float | integer]`(对于field), `tag_key <operator> ['tag_value']`(对于tag)

supported operators: `= <> != > >= < <=`

## group
Syntax: `SELECT_clause FROM_clause [WHERE_clause] GROUP BY [* | <tag_key>[,<tag_key]]`

### group by time intervals
Syntax: `SELECT <function>(<field_key>) FROM_clause WHERE <time_range> GROUP BY time(<time_interval>),[tag_key] [fill(<fill_option>)]`

fill_option:
* `Any`: numerical value: Reports the given numerical value for time intervals with no data. 
* `linear`: Reports the results of linear interpolation for time intervals with no data. 
* `none`: Reports no timestamp and no value for time intervals with no data. 
* `null`: Reports null for time intervals with no data but returns a timestamp. This is the same as the default behavior. 
* `previous`: Reports the value from the previous time interval for time intervals with no data.

## order 
### order by time desc
Syntax: `SELECT_clause [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] ORDER BY time DESC`
* order by 必须在 where 和 group by 之后.


## limit
Syntax: `SELECT_clause [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] LIMIT <N>`

