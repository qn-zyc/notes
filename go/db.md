# 参考资料

* <https://github.com/golang/go/wiki/SQLInterface>


# 连接

## mysql

```
name:password@tcp(ip:3306)/dbname?charset=utf8&loc=Local
```

ipv6:

```
user:password@tcp([hostname]:port)/dbname
```


# 从数据库中解析datetime类型的字段

连接时添加 `parseTime=true` 选项，接受字段类型为 `*time.Time`.


# SQLInterface

> https://github.com/golang/go/wiki/SQLInterface

## Introduction ##

The database/sql package provides a generic interface around SQL (or SQL-like) databases. See the [official documentation](http://golang.org/pkg/database/sql/) for details.

This page provides example usage patterns.

## Database driver ##

The database/sql package must be used in conjunction with a database driver. See [http://golang.org/s/sqldrivers](http://golang.org/s/sqldrivers) for a list of drivers.

The documentation below assumes a driver has been imported.

## Connecting to a database ##

Open is used to create a database handle:

```go
db, err := sql.Open(driver, dataSourceName)
```

Where driver specifies a database driver and dataSourceName specifies database-specific connection information such as database name and authentication credentials.

Note that Open does not directly open a database connection: this is deferred until a query is made. To verify that a connection can be made before making a query, use the Ping function:

```go
if err := db.Ping(); err != nil {
  log.Fatal(err)
}
```

After use, the database is closed using Close.

## Executing queries ##

Exec is used for queries where no rows are returned:

```go
result, err := db.Exec(
    "INSERT INTO users (name, age) VALUES ($1, $2)",
    "gopher",
    27,
)
```

Where result contains the last insert ID and number of rows affected. The availability of these values is dependent on the database driver.

Query is used for retrieval:

```go
rows, err := db.Query("SELECT name FROM users WHERE age = $1", age)
if err != nil {
    log.Fatal(err)
}
for rows.Next() {
    var name string
    if err := rows.Scan(&name); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%s is %d\n", name, age)
}
if err := rows.Err(); err != nil {
    log.Fatal(err)
}
```

QueryRow is used where only a single row is expected:

```go
var age int64
row := db.QueryRow("SELECT age FROM users WHERE name = $1", name)
err := row.Scan(&age)
```

Prepared statements can be created with Prepare:

```go
age := 27
stmt, err := db.Prepare("SELECT name FROM users WHERE age = $1")
if err != nil {
    log.Fatal(err)
}
rows, err := stmt.Query(age)
// process rows
```

Exec, Query and QueryRow can be called on statements. After use, a statement should be closed with Close.

## Transactions ##

Transactions are started with Begin:

```
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
```

The Exec, Query, QueryRow and Prepare functions already covered can be used in a transaction.

A transaction must end with a call to Commit or Rollback.

## Dealing with NULL ##

If a database column is nullable, one of the types supporting null values should be passed to Scan.

For example, if the name column in the names table is nullable:

```go
var name NullString
err := db.QueryRow("SELECT name FROM names WHERE id = $1", id).Scan(&name)
...
if name.Valid {
    // use name.String
} else {
    // value is NULL
}
```

Only NullBool, NullFloat64, NullInt64 and NullString are implemented in database/sql. Implementations of database-specific null types are left to the database driver.




# 批量操作各种方式效率分析 #

```go
package main

import (
    "strconv"
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
    "fmt"
    "time"
    "log"
)

var db = &sql.DB{}

func init(){
    db,_ = sql.Open("mysql", "root:root@/book")
} 

func main() {
    insert()
    query()
    update()
    query()
    delete()
}

func update(){
    //方式1 update
    start := time.Now()
    for i := 1001;i<=1100;i++{
        db.Exec("UPdate user set age=? where uid=? ",i,i)
    }
    end := time.Now()
    fmt.Println("方式1 update total time:",end.Sub(start).Seconds())
    
    //方式2 update
    start = time.Now()
    for i := 1101;i<=1200;i++{
        stm,_ := db.Prepare("UPdate user set age=? where uid=? ")
        stm.Exec(i,i)
        stm.Close()
    }
    end = time.Now()
    fmt.Println("方式2 update total time:",end.Sub(start).Seconds())
    
    //方式3 update
    start = time.Now()
    stm,_ := db.Prepare("UPdate user set age=? where uid=?")
    for i := 1201;i<=1300;i++{
        stm.Exec(i,i)
    }
    stm.Close()
    end = time.Now()
    fmt.Println("方式3 update total time:",end.Sub(start).Seconds())
    
    //方式4 update
    start = time.Now()
    tx,_ := db.Begin()
    for i := 1301;i<=1400;i++{
        tx.Exec("UPdate user set age=? where uid=?",i,i)
    }
    tx.Commit()
    
    end = time.Now()
    fmt.Println("方式4 update total time:",end.Sub(start).Seconds())
    
    //方式5 update
    start = time.Now()
    for i := 1401;i<=1500;i++{
        tx,_ := db.Begin()
        tx.Exec("UPdate user set age=? where uid=?",i,i)
        tx.Commit()
    }
    end = time.Now()
    fmt.Println("方式5 update total time:",end.Sub(start).Seconds())
    
    // 方式1 update total time: 7.3394198
	// 方式2 update total time: 7.8464488
	// 方式3 update total time: 6.0053435
	// 方式4 update total time: 0.6630379000000001
	// 方式5 update total time: 4.5402597
    
}

func delete(){
    //方式1 delete
    start := time.Now()
    for i := 1001;i<=1100;i++{
        db.Exec("DELETE FROM USER WHERE uid=?",i)
    }
    end := time.Now()
    fmt.Println("方式1 delete total time:",end.Sub(start).Seconds())
    
    //方式2 delete
    start = time.Now()
    for i := 1101;i<=1200;i++{
        stm,_ := db.Prepare("DELETE FROM USER WHERE uid=?")
        stm.Exec(i)
        stm.Close()
    }
    end = time.Now()
    fmt.Println("方式2 delete total time:",end.Sub(start).Seconds())
    
    //方式3 delete
    start = time.Now()
    stm,_ := db.Prepare("DELETE FROM USER WHERE uid=?")
    for i := 1201;i<=1300;i++{
        stm.Exec(i)
    }
    stm.Close()
    end = time.Now()
    fmt.Println("方式3 delete total time:",end.Sub(start).Seconds())
    
    //方式4 delete
    start = time.Now()
    tx,_ := db.Begin()
    for i := 1301;i<=1400;i++{
        tx.Exec("DELETE FROM USER WHERE uid=?",i)
    }
    tx.Commit()
    
    end = time.Now()
    fmt.Println("方式4 delete total time:",end.Sub(start).Seconds())
    
    //方式5 delete
    start = time.Now()
    for i := 1401;i<=1500;i++{
        tx,_ := db.Begin()
        tx.Exec("DELETE FROM USER WHERE uid=?",i)
        tx.Commit()
    }
    end = time.Now()
    fmt.Println("方式5 delete total time:",end.Sub(start).Seconds())
    
    // 方式1 delete total time: 3.8652211000000003
	// 方式2 delete total time: 3.8582207
	// 方式3 delete total time: 3.6972114
	// 方式4 delete total time: 0.43202470000000004
	// 方式5 delete total time: 3.7972172
    
}

func query(){
    
    //方式1 query
    start := time.Now()
    rows,_ := db.Query("SELECT uid,username FROM USER")
    defer rows.Close()
    for rows.Next(){
         var name string
         var id int
        if err := rows.Scan(&id,&name); err != nil {
            log.Fatal(err)
        }
        //fmt.Printf("name:%s ,id:is %d\n", name, id)
    }
    end := time.Now()
    fmt.Println("方式1 query total time:",end.Sub(start).Seconds())
    
    //方式2 query
    start = time.Now()
    stm,_ := db.Prepare("SELECT uid,username FROM USER")
    defer stm.Close()
    rows,_ = stm.Query()
    defer rows.Close()
    for rows.Next(){
         var name string
         var id int
        if err := rows.Scan(&id,&name); err != nil {
            log.Fatal(err)
        }
       // fmt.Printf("name:%s ,id:is %d\n", name, id)
    }
    end = time.Now()
    fmt.Println("方式2 query total time:",end.Sub(start).Seconds())
    
    
    //方式3 query
    start = time.Now()
    tx,_ := db.Begin()
    defer tx.Commit()
    rows,_ = tx.Query("SELECT uid,username FROM USER")
    defer rows.Close()
    for rows.Next(){
         var name string
         var id int
        if err := rows.Scan(&id,&name); err != nil {
            log.Fatal(err)
        }
        //fmt.Printf("name:%s ,id:is %d\n", name, id)
    }
    end = time.Now()
    fmt.Println("方式3 query total time:",end.Sub(start).Seconds())
    
	// 方式1 query total time: 0.0070004
	// 方式2 query total time: 0.0100006
	// 方式3 query total time: 0.0100006
}

func insert() {
    
    //方式1 insert
    //strconv,int转string:strconv.Itoa(i)
    start := time.Now()
    for i := 1001;i<=1100;i++{
        //每次循环内部都会去连接池获取一个新的连接，效率低下
        db.Exec("INSERT INTO user(uid,username,age) values(?,?,?)",i,"user"+strconv.Itoa(i),i-1000)
    }
    end := time.Now()
    fmt.Println("方式1 insert total time:",end.Sub(start).Seconds())
    
    //方式2 insert
    start = time.Now()
    for i := 1101;i<=1200;i++{
        //Prepare函数每次循环内部都会去连接池获取一个新的连接，效率低下
        stm,_ := db.Prepare("INSERT INTO user(uid,username,age) values(?,?,?)")
        stm.Exec(i,"user"+strconv.Itoa(i),i-1000)
        stm.Close()
    }
    end = time.Now()
    fmt.Println("方式2 insert total time:",end.Sub(start).Seconds())
    
    //方式3 insert
    start = time.Now()
    stm,_ := db.Prepare("INSERT INTO user(uid,username,age) values(?,?,?)")
    for i := 1201;i<=1300;i++{
        //Exec内部并没有去获取连接，为什么效率还是低呢？
        stm.Exec(i,"user"+strconv.Itoa(i),i-1000)
    }
    stm.Close()
    end = time.Now()
    fmt.Println("方式3 insert total time:",end.Sub(start).Seconds())
    
    //方式4 insert
    start = time.Now()
    //Begin函数内部会去获取连接
    tx,_ := db.Begin()
    for i := 1301;i<=1400;i++{
        //每次循环用的都是tx内部的连接，没有新建连接，效率高
        tx.Exec("INSERT INTO user(uid,username,age) values(?,?,?)",i,"user"+strconv.Itoa(i),i-1000)
    }
    //最后释放tx内部的连接
    tx.Commit()
    
    end = time.Now()
    fmt.Println("方式4 insert total time:",end.Sub(start).Seconds())
    
    //方式5 insert
    start = time.Now()
    for i := 1401;i<=1500;i++{
        //Begin函数每次循环内部都会去连接池获取一个新的连接，效率低下
        tx,_ := db.Begin()
        tx.Exec("INSERT INTO user(uid,username,age) values(?,?,?)",i,"user"+strconv.Itoa(i),i-1000)
        //Commit执行后连接也释放了
        tx.Commit()
    }
    end = time.Now()
    fmt.Println("方式5 insert total time:",end.Sub(start).Seconds())
    
    // 方式1 insert total time: 3.7952171
	// 方式2 insert total time: 4.3162468
	// 方式3 insert total time: 4.3392482
	// 方式4 insert total time: 0.3970227
	// 方式5 insert total time: 7.3894226
}
```





# database/sql接口
Go没有官方提供数据库驱动，而是为开发者开发数据库驱动定义了一些标准接口。

## sql.Register
这个存在于database/sql的函数是用来注册数据库驱动的，当第三方开发者开发数据库驱动时，都会实现init函数，在init里面会调用这个`Register(name string, driver driver.Driver)`完成本驱动的注册。

我们来看一下mymysql、sqlite3的驱动里面都是怎么调用的：

```go
//https://github.com/mattn/go-sqlite3驱动
func init() {
    sql.Register("sqlite3", &SQLiteDriver{})
}

//https://github.com/mikespook/mymysql驱动
// Driver automatically registered in database/sql
var d = Driver{proto: "tcp", raddr: "127.0.0.1:3306"}
func init() {
    Register("SET NAMES utf8")
    sql.Register("mymysql", &d)
}
```

在database/sql内部通过一个map来存储用户定义的相应驱动。

```go
var drivers = make(map[string]driver.Driver)
drivers[name] = driver
```

引用第三方库示例：

```go
  import (
      "database/sql"
      _ "github.com/mattn/go-sqlite3"
  )
```
  
其中，`_` 就是为了调用包的init函数。

## driver.Driver

Driver是一个数据库驱动的接口，他定义了一个method： `Open(name string)`，这个方法返回一个数据库的Conn接口。

```go
type Driver interface {
    Open(name string) (Conn, error)
}
```

返回的Conn只能用来进行一次goroutine的操作，也就是说**不能把这个Conn应用于Go的多个goroutine里面**。如下代码会出现错误

```go
// ...
go goroutineA (Conn)  //执行查询操作
go goroutineB (Conn)  //执行插入操作
// ...
```

上面这样的代码可能会使Go不知道某个操作究竟是由哪个goroutine发起的,从而导致数据混乱，比如可能会把goroutineA里面执行的查询操作的结果返回给goroutineB从而使B错误地把此结果当成自己执行的插入数据。

第三方驱动都会定义这个函数，它会解析name参数来获取相关数据库的连接信息，解析完成后，它将使用此信息来初始化一个Conn并返回它。

## driver.Conn
Conn是一个数据库连接的接口定义，他定义了一系列方法，这个Conn只能应用在一个goroutine里面，不能使用在多个goroutine里面。

```go
type Conn interface {
    Prepare(query string) (Stmt, error)
    Close() error
    Begin() (Tx, error)
}
```

Prepare函数返回与当前连接相关的执行Sql语句的准备状态，可以进行查询、删除等操作。

Close函数关闭当前的连接，执行释放连接拥有的资源等清理工作。因为驱动实现了database/sql里面建议的conn pool，所以你不用再去实现缓存conn之类的，这样会容易引起问题。

Begin函数返回一个代表事务处理的Tx，通过它你可以进行查询,更新等操作，或者对事务进行回滚、递交。

## driver.Stmt

Stmt是一种准备好的状态，和Conn相关联，而且只能应用于一个goroutine中，不能应用于多个goroutine。

```go
type Stmt interface {
    Close() error
    NumInput() int
    Exec(args []Value) (Result, error)
    Query(args []Value) (Rows, error)
}
```

Close函数关闭当前的链接状态，但是如果当前正在执行query，query还是有效返回rows数据。

NumInput函数返回当前预留参数的个数，当返回>=0时数据库驱动就会智能检查调用者的参数。当数据库驱动包不知道预留参数的时候，返回-1。

Exec函数执行Prepare准备好的sql，传入参数执行update/insert等操作，返回Result数据

Query函数执行Prepare准备好的sql，传入需要的参数执行select操作，返回Rows结果集。

## driver.Tx

事务处理一般就两个过程，递交或者回滚。数据库驱动里面也只需要实现这两个函数就可以

```go
type Tx interface {
    Commit() error
    Rollback() error
}
```

这两个函数一个用来递交一个事务，一个用来回滚事务。

## driver.Execer
这是一个Conn可选择实现的接口

```go
type Execer interface {
    Exec(query string, args []Value) (Result, error)
}
```

如果这个接口没有定义，那么在调用DB.Exec,就会首先调用Prepare返回Stmt，然后执行Stmt的Exec，然后关闭Stmt。

## driver.Result
这个是执行Update/Insert等操作返回的结果接口定义

```go
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```

LastInsertId函数返回由数据库执行插入操作得到的自增ID号。

RowsAffected函数返回query操作影响的数据条目数。

## driver.Rows
Rows是执行查询返回的结果集接口定义

```go
type Rows interface {
    Columns() []string
    Close() error
    Next(dest []Value) error
}
```

Columns函数返回查询数据库表的字段信息，这个返回的slice和sql查询的字段一一对应，而不是返回整个表的所有字段。

Close函数用来关闭Rows迭代器。

Next函数用来返回下一条数据，把数据赋值给dest。dest里面的元素必须是driver.Value的值除了string，返回的数据里面所有的string都必须要转换成[]byte。如果最后没数据了，Next函数最后返回io.EOF。

## driver.RowsAffected
RowsAffected其实就是一个int64的别名，但是他实现了Result接口，用来底层实现Result的表示方式

```go
type RowsAffected int64
func (RowsAffected) LastInsertId() (int64, error)
func (v RowsAffected) RowsAffected() (int64, error)
```

## driver.Value
Value其实就是一个空接口，他可以容纳任何的数据

```go
type Value interface{}
```

drive的Value是驱动必须能够操作的Value，Value要么是nil，要么是下面的任意一种

- int64
- float64
- bool
- []byte
- string   [*]除了Rows.Next返回的不能是string.
- time.Time

## driver.ValueConverter
ValueConverter接口定义了如何把一个普通的值转化成driver.Value的接口

```go
type ValueConverter interface {
    ConvertValue(v interface{}) (Value, error)
}
```

在开发的数据库驱动包里面实现这个接口的函数在很多地方会使用到，这个ValueConverter有很多好处：

- 转化driver.value到数据库表相应的字段，例如int64的数据如何转化成数据库表uint16字段
- 把数据库查询结果转化成driver.Value值
- 在scan函数里面如何把driver.Value值转化成用户定义的值

## driver.Valuer
Valuer接口定义了返回一个driver.Value的方式

```go
type Valuer interface {
    Value() (Value, error)
}
```

很多类型都实现了这个Value方法，用来自身与driver.Value的转化。

## database/sql
database/sql在database/sql/driver提供的接口基础上定义了一些更高阶的方法，用以简化数据库操作,同时内部还建议性地实现一个conn pool。

```go
type DB struct {
    driver   driver.Driver
    dsn      string
    mu       sync.Mutex // protects freeConn and closed
    freeConn []driver.Conn
    closed   bool
}
```

我们可以看到Open函数返回的是DB对象，里面有一个freeConn，它就是那个简易的连接池。它的实现相当简单或者说简陋，就是当执行Db.prepare的时候会defer db.putConn(ci, err),也就是把这个连接放入连接池，每次调用conn的时候会先判断freeConn的长度是否大于0，大于0说明有可以复用的conn，直接拿出来用就是了，如果不大于0，则创建一个conn,然后再返回之。


