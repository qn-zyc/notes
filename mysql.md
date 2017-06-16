<!-- TOC -->

- [1. 参考资料](#1-参考资料)
- [2. 数据库操作](#2-数据库操作)
    - [2.1. 安装](#21-安装)
        - [2.1.1. 重设ROOT初始密码](#211-重设root初始密码)
        - [2.1.2. 安装时的初始密码](#212-安装时的初始密码)
    - [2.2. 连接](#22-连接)
    - [2.3. DATABASE](#23-database)
        - [2.3.1. 显示数据库](#231-显示数据库)
        - [2.3.2. 创建数据库](#232-创建数据库)
    - [2.4. USER](#24-user)
        - [2.4.1. 创建用户](#241-创建用户)
        - [2.4.2. 授权](#242-授权)
            - [2.4.2.1. 权限列表](#2421-权限列表)
        - [2.4.3. 显示用户](#243-显示用户)
        - [2.4.4. 修改用户密码](#244-修改用户密码)
        - [2.4.5. 创建用户无法在本地登录](#245-创建用户无法在本地登录)
    - [2.5. 字段类型](#25-字段类型)
        - [2.5.1. 数字列类型](#251-数字列类型)
        - [2.5.2. 字符串列类型](#252-字符串列类型)
        - [2.5.3. 日期和时间列类型](#253-日期和时间列类型)
    - [2.6. 表](#26-表)
        - [2.6.1. 查看表状态](#261-查看表状态)
        - [2.6.2. 创建表](#262-创建表)
        - [2.6.3. 复制表结构](#263-复制表结构)
        - [2.6.4. 显示表](#264-显示表)
        - [2.6.5. 查看表结构](#265-查看表结构)
        - [2.6.6. 查看创建表的语句](#266-查看创建表的语句)
        - [2.6.7. 重命名表](#267-重命名表)
        - [2.6.8. 查看字段详细信息](#268-查看字段详细信息)
        - [2.6.9. 增加字段](#269-增加字段)
        - [2.6.10. 删除字段](#2610-删除字段)
        - [2.6.11. 修改字段](#2611-修改字段)
            - [2.6.11.1. 修改字段的字符集](#26111-修改字段的字符集)
        - [2.6.12. 插入数据](#2612-插入数据)
        - [2.6.13. 存在时更新，否则插入](#2613-存在时更新否则插入)
            - [2.6.13.1. on duplicate key update](#26131-on-duplicate-key-update)
            - [2.6.13.2. replace into](#26132-replace-into)
        - [2.6.14. 查看外键](#2614-查看外键)
    - [2.7. 添加外键](#27-添加外键)
        - [2.7.1. 创建表时指定外键](#271-创建表时指定外键)
    - [2.8. 查询](#28-查询)
        - [2.8.1. limit](#281-limit)
    - [2.9. 导入与导出](#29-导入与导出)
        - [2.9.1. 导入sql文件](#291-导入sql文件)
        - [2.9.2. 导出特定表结构和数据](#292-导出特定表结构和数据)

<!-- /TOC -->

# 1. 参考资料

- MySQL 加锁处理分析 <http://hedengcheng.com/?p=771>
- 官方文档 <http://dev.mysql.com/doc/refman/5.7/en/>


# 2. 数据库操作

## 2.1. 安装

### 2.1.1. 重设ROOT初始密码

报错信息：

```bash
RROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

解决方案：

```bash
SET PASSWORD=PASSWORD('newpassword');
```



### 2.1.2. 安装时的初始密码

通过 `ps aux | grep mysql` 找到 error log 的位置，比如：

```
--log-error=/var/log/mysqld.log
```

在里面搜索 `temporary password`



## 2.2. 连接

本地

```bash
mysql -u root -p
```

远程

```bash
mysql -h110.110.110.110 -u root -p 123
```

使用数据库：

```bash
use 数据库名;
```

查询主机名和端口：

```bash
select @@global.hostname, @@global.port;
```


## 2.3. DATABASE ##

### 2.3.1. 显示数据库

```bash
show databases;
```

### 2.3.2. 创建数据库

```bash
create database mysql_demo;
```

## 2.4. USER ##

### 2.4.1. 创建用户

```bash
CREATE USER username IDENTIFIED BY 'password';
```

上面建立的用户可以在任何地方登陆。

如果要限制在固定地址登陆，比如localhost 登陆：

```bash
CREATE USER username@localhost IDENTIFIED BY 'password';
```

### 2.4.2. 授权

```bash
GRANT privileges ON databasename.tablename TO username;
```

* privileges——用户的操作权限,如SELECT , INSERT , UPDATE 等，如果要授予所有的权限则使用ALL.
* databasename——数据库名
* tablename——表名,如果要授予该用户对所有数据库和表的相应操作权限则可用 `*` 表示, 如 `*.*`.

授权后还要刷新系统权限表

```bash
flush privileges;
```

查询当前用户的授权：

```
show /* gh-ost */ grants for current_user();
```

为root用户在本地登录时赋予所有权限：

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'127.0.0.1' IDENTIFIED BY 'password' WITH GRANT OPTION;
flush privileges;
```



#### 2.4.2.1. 权限列表

- SELECT
- UPDATE
- DELETE
- ALTER
- DROP
- REFERENCES 指定外键的时候需要，v5.6.22之前不需要。





### 2.4.3. 显示用户 ###

```bash
select host,user from mysql.user;
```

### 2.4.4. 修改用户密码

```bash
set password for 'root'@'localhost' = password('tyuhbnm');
flush privileges;
```

### 2.4.5. 创建用户无法在本地登录

创建语句如下：`grant select,insert,update,delete on test.* to appadmin@"%" identified by "password";` 其中@“%”是可以在任何地址登录。
但是登录失败，原因是mysql.user中有一条host是localhost而user为空的记录，这是个匿名用户。

解决方案：
`grant select,insert,update,delete on test.* to appadmin@"localhost" identified by "password"; `





## 2.5. 字段类型

### 2.5.1. 数字列类型 ###

数字列类型主要分为两种：整数型和浮点型。

所有的数字列类型都允许有两个选项：UNSIGNED和ZEROFILL。选择UNSIGNED的列不允许有负数，选择了ZEROFILL的列会为数值添加零。下面是MySQL中可用的数字列类型

* TINYINT——一个微小的整数，支持 -128到127(SIGNED)，0到255(UNSIGNED)，需要1个字节存储
* BIT——同TINYINT(1)
* BOOL——同TINYINT(1)
* SMALLINT——一个小整数，支持 -32768到32767(SIGNED)，0到65535(UNSIGNED)，需要2个字节存储
* MEDIUMINT——一个中等整数，支持 -8388608到8388607(SIGNED)，0到16777215(UNSIGNED)，需要3个字节存储
* INT——一个整数，支持 -2147493648到2147493647(SIGNED)，0到4294967295(UNSIGNED)，需要4个字节存储
* INTEGER——同INT
* BIGINT——一个大整数，支持 -9223372036854775808到9223372036854775807(SIGNED)，0到18446744073709551615(UNSIGNED)，需要8个字节存储
* FLOAT(precision)——一个浮点数。precision<=24用于单精度浮点数；precision在25和53之间，用于双精度浮点数。FLOAT(X)与相诮的FLOAT和DOUBLE类型有差相同的范围，但是没有定义显示尺寸和小数位数。在MySQL3.23之前，这不是一个真的浮点值，且总是有两位小数。MySQL中的所有计算都用双精度，所以这会带来一些意想不到的问题。
* FLOAT——一个小的菜单精度浮点数。支持 -3.402823466E+38到-1.175494351E-38，0和1.175494351E-38 to 3.402823466E+38，需要4个字节存储。如果是UNSIGNED，正数的范围保持不变，但负数是不允许的。
* DOUBLE——一个双精度浮点数。支持 -1.7976931348623157E+308到-2.2250738585072014E-308，0和2.2250738585072014E-308到1.7976931348623157E+308。如果是FLOAT，UNSIGNED不会改变正数范围，但负数是不允许的。
* DOUBLE PRECISION——同DOUBLE
* REAL——同DOUBLE
* DECIMAL——将一个数像字符串那样存储，每个字符占一个字节
* DEC——同DECIMAL
* NUMERIC——同DECIMAL

### 2.5.2. 字符串列类型

* CHAR——字符。固定长度的字串，在右边补齐空格，达到指定的长度。支持从0到155个字符。搜索值时，后缀的空格将被删除。
* VARCHAR——可变长的字符。一个可变长度的字串，其中的后缀空格在存储值时被删除。支持从0到255字符
* TINYBLOB——微小的二进制对象。支持255个字符。需要长度+1字节的存储。与TINYTEXT一样，只不过搜索时是区分大小写的。(0.25KB)
* TINYTEXT——支持255个字符。要求长度+1字节的存储。与TINYBLOB一样，只不过搜索时会忽略大小写。(0.25KB)
* BLOB——二进制对象。支持65535个字符。需要长度+2字节的存储。 (64KB)
* TEXT——支持65535个字符。要求长度+2字节的存储。 (64KB)
* MEDIUMBLOB——中等大小的二进制对象。支持16777215个字符。需要长度+3字节的存储。 (16M)
* MEDIUMTEXT——支持16777215个字符。需要长度+3字节的存储。 (16M)
* LONGBLOB——大的的二进制对象。支持4294967295个字符。需要长度+4字节的存储。 (4G)
* LONGTEXT——支持4294967295个字符。需要长度+4字节的存储。(4G)
* ENUM——枚举。只能有一个指定的值，即NULL或""，最大有65535个值
* SET——一个集合。可以有0到64个值，均来自于指定清单

### 2.5.3. 日期和时间列类型

日期和时间列类型用于处理时间数据，可以存储当日的时间或出生日期这样的数据。格式的规定：Y表示年、M（前M）表示月、D表示日、H表示小时、M（后M）表示分钟、S表示秒。下面是MySQL中可用的日期和时间列类型

* DATETIME——格式：'YYYY-MM-DD HH:MM:SS'，范围：'1000-01-01 00:00:00'到'9999-12-31 23:59:59'
* DATE——格式：'YYYY-MM-DD'，范围：'1000-01-01'到'9999-12-31'
* TIMESTAMP——格式：'YYYYMMDDHHMMSS'、'YYMMDDHHMMSS'、'YYYYMMDD'、'YYMMDD'，范围：'1970-01-01 00:00:00'到'2037-01-01 00:00:00'
* TIME——格式：'HH:MM:SS'
* YEAR——格式：'YYYY，范围：'1901'到'2155'



## 2.6. 表

### 2.6.1. 查看表状态

```bash
show table status from gewara like '%address%';
```


### 2.6.2. 创建表

```bash
mysql> create table IF NOT EXISTS table1 (
    -> id int(4) not null primary key auto_increment,
    -> name char(20) not null,
    -> sex bool not null default 0,
    -> degree double(10,2)
    -> );
```

创建表时指定字符集：

```bash
create table (...) default charset=utf8;
```

创建表时指定表和字段的注释：

```bash
CREATE TABLE IF NOT EXISTS send_record ( 
		id bigint not null primary key auto_increment comment '主键', 
		uuid char(40) not null comment '备用，迁移时做外键', 
		phone_number char(20) comment '手机号', 
		content varchar(1000) comment '短信内容', 
		msg_kind varchar(20) comment '消息类型，可能的值有auth,notice,goupiao,normal,inside,marketing，后续可能会添加', 
		ret int comment '返回码', 
		msg varchar(200) comment '返回信息', 
		spid varchar(20) comment '服务商ID,当前可选值有yunpian,winner_look,tencent,gaopeng,后续可能会增加', 
		spret int comment '服务商返回码', 
		batchid int comment '服务商返回的批次号', 
		format_date varchar(20) comment '格式化后的发送短信的日期', 
		format_time varchar(20) comment '格式化后的发送短信的时间', 
		submitter varchar(50) comment '提交者，即client_id' 
) default charset=utf8 comment '短信发送记录表';
```

从已存在表创建并复制数据：

```bash
create table newTable select * from oldTable;
```


### 2.6.3. 复制表结构

```bash
create table newTable like oldTable;
```

或者：

```bash
create table newTable select * from oldTable limit 0;
```

### 2.6.4. 显示表

```bash
show tables;
```

### 2.6.5. 查看表结构

```bash
desc TABLE_NAME;
```

或者：

```bash
SHOW COLUMNS FROM table_name;
```

Key那一栏，可能会有4种值，即 ' '，'PRI'，'UNI'，'MUL'。

* 如果Key是空的，那么该列值的可以重复，表示该列没有索引，或者是一个非唯一的复合索引的非前导列；
* 如果Key是PRI，那么该列是主键的组成部分；
* 如果Key是UNI，那么该列是一个唯一值索引的第一列（前导列），并别不能含有空值（NULL）；
* 如果Key是MUL，那么该列的值可以重复，该列是一个非唯一索引的前导列（第一列）或者是一个唯一性索引的组成部分但是可以含有空值NULL。

如果对于一个列的定义，同时满足上述4种情况的多种，比如一个列既是PRI，又是UNI，那么"desc 表名"的时候，显示的Key值按照优先级来显示 PRI->UNI->MUL。那么此时，显示PRI。

一个唯一性索引列可以显示为PRI，并且该列不能含有空值，同时该表没有主键。

一个唯一性索引列可以显示为MUL，如果多列构成了一个唯一性复合索引，因为虽然索引的多列组合是唯一的，比如ID+NAME是唯一的，但是没一个单独的列依然可以有重复的值，只要ID+NAME是唯一的即可。

### 2.6.6. 查看创建表的语句

```bash
show create table send_record;
```

### 2.6.7. 重命名表 ###

<http://dev.mysql.com/doc/refman/5.7/en/rename-table.html>

```bash
RENAME TABLE tbl_name TO new_tbl_name [, tbl_name2 TO new_tbl_name2] ...
```

```bash
RENAME TABLE old_table TO new_table;
等同于
ALTER TABLE old_table RENAME new_table;
```

### 2.6.8. 查看字段详细信息

```sql
show full columns from 表名;
```

### 2.6.9. 增加字段

```sql
alter table directory add column index_url varchar(256) default null comment '章节书目链接' after dir_url;
```

### 2.6.10. 删除字段

```sql
alter table user DROP COLUMN new2;
```



### 2.6.11. 修改字段

```sql
alter table user MODIFY new1 VARCHAR(10); 　　　　　　　　　　 //修改一个字段的类型
alter table user CHANGE new1 new4 int;　　　　　　　　　　　　　 //修改一个字段的名称，此时一定要重新指定该字段的类型
```

#### 2.6.11.1. 修改字段的字符集

```bash
alter table 表名 change 字段名 字段名 varchar(45) character set utf8;
```


### 2.6.12. 插入数据

```mysql
insert into table1(name, sex, degree) values('chen', 1, 123.45);
```

批量插入：

```mysql
INSERT INTO table (field1,field2,field3) VALUES ('a',"b","c"), ('a',"b","c"),('a',"b","c");
```



### 2.6.13. 存在时更新，否则插入

#### 2.6.13.1. on duplicate key update

```sql
insert into users(id, name, age) values(2, "chen", 27) on duplicate key update age=values(age), name=values(name);
```

id是主键，当存在id=2的记录时更新age和name值，否则插入一条新的记录。

插入多行记录:

```sql
insert into users(id, name, age) values(2, "chen", 27), (3, "chen2", 28) on duplicate key update age=values(age), name=values(name);
```

文档： <http://dev.mysql.com/doc/refman/5.7/en/insert-on-duplicate.html>


#### 2.6.13.2. replace into

```sql
replace into users(id, name, age) values(7, "chen7", 30);
```

这个和 `on duplicate key update` 的区别是`on duplicate key update`可以控制更新的字段，而`replace into`更新的是所有指定的字段。

文档： <http://dev.mysql.com/doc/refman/5.7/en/replace.html>


### 2.6.14. 查看外键

```bash
SELECT 
    TABLE_SCHEMA, TABLE_NAME
FROM
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE
    REFERENCED_TABLE_NAME IS NOT NULL
        AND ((TABLE_SCHEMA = '' AND TABLE_NAME = '')
        OR (REFERENCED_TABLE_SCHEMA = ''
        AND REFERENCED_TABLE_NAME = ''));
```



## 2.7. 添加外键

```
ALTER TABLE yourtablename
ADD [CONSTRAINT 外键名] FOREIGN KEY [id] (index_col_name, …)
REFERENCES tbl_name (index_col_name, …)
[ON DELETE {CASCADE | SET NULL | NO ACTION | RESTRICT}]
[ON UPDATE {CASCADE | SET NULL | NO ACTION | RESTRICT}]
```

例子：

```
alter table child add constraint fk_1 foreign key (parent_id) references
parent(id) on update restrict on delete set null;
```

说明:
on delete/on update,用于定义delete,update操作.以下是update,delete操作的各种约束类型:
CASCADE:
外键表中外键字段值会被更新,或所在的列会被删除.
RESTRICT:
RESTRICT也相当于no action,即不进行任何操作.即,拒绝父表update外键关联列,delete记录.
set null:
被父面的外键关联字段被update ,delete时,子表的外键列被设置为null.
而对于insert,子表的外键列输入的值,只能是父表外键关联列已有的值.否则出错.



### 2.7.1. 创建表时指定外键

```mysql
create table bench_sending (
    id int not null primary key auto_increment,
    name varchar(100) default null,
    sms_content varchar(1000) not null,
    pkg_id int not null,
    test_phone varchar(11) default null,
    send_type int default 0 not null,
    send_time varchar(30) default null,
    constraint foreign key (pkg_id)
        references phone_pkg (id)
        on delete cascade on update cascade
)
```



## 2.8. 查询

### 2.8.1. limit

```sql
SELECT * FROM table LIMIT [offset,] rows | rows OFFSET offset  
```

```sql
mysql> SELECT * FROM table LIMIT 5,10; // 检索记录行 6-15  
  
//为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1：   
mysql> SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last.  
  
//如果只给定一个参数，它表示返回最大的记录行数目：   
mysql> SELECT * FROM table LIMIT 5; //检索前 5 个记录行  
  
//换句话说，LIMIT n 等价于 LIMIT 0,n。  
```

最好用上 `order by`

形式1：

```sql
SELECT * FROM articles WHERE category_id = 123 ORDER BY id LIMIT 50, 10  
```

形式2，使用子查询提高查询效率：

```sql
SELECT * FROM articles WHERE  id >=  
 (SELECT id FROM articles  WHERE category_id = 123 ORDER BY id LIMIT 10000, 1) LIMIT 10  
```


## 2.9. 导入与导出

### 2.9.1. 导入sql文件

```bash
mysql -u root -p 数据库名 < sql文件路径
```

### 2.9.2. 导出特定表结构和数据

```bash
mysqldump -h localhost -uroot -p 数据库名 表名 > /path/filename.sql
```

导出的文件中大概会做以下几件事情：
1. drop table
2. create table
3. lock table
4. insert values



