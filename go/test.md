<!-- TOC -->

- [测试用例](#测试用例)
    - [代码覆盖率](#代码覆盖率)
    - [指定超时时间](#指定超时时间)
- [压力测试](#压力测试)
    - [记录cpu占用](#记录cpu占用)
    - [记录内存分配](#记录内存分配)
    - [并发测试](#并发测试)
- [例子用例](#例子用例)
- [测试脚本](#测试脚本)
    - [除vendor外的测试](#除vendor外的测试)
- [测试技巧](#测试技巧)
    - [子测试](#子测试)
    - [表格驱动](#表格驱动)
    - [测试用例的当前目录](#测试用例的当前目录)
    - [flag](#flag)
- [参考](#参考)

<!-- /TOC -->


# 测试用例

- 文件名必须是`_test.go`结尾的，这样在执行`go test`的时候才会执行到相应的代码
- 你必须`import testing`这个包
- 所有的测试用例函数必须是`Test`开头
- 测试用例会按照源代码中写的顺序依次执行
- 测试函数`TestXxx()`的参数是`testing.T`，我们可以使用该类型来记录错误或者是测试状态
- 测试格式：`func TestXxx (t *testing.T)`,Xxx部分可以为任意的字母数字的组合，但是首字母不能是小写字母[a-z]，例如`Testintdiv`是错误的函数名。
- 函数中通过调用`testing.T`的Error, Errorf, FailNow, Fatal, FatalIf方法，说明测试不通过，调用Log方法用来记录测试的信息。
- 执行当前目录下所有的单元测试: `go test -v ./...`.


```bash
go test -v -run=函数名正则 包路径
```

```go
package gotest

import (
    "testing"
)

func Test_Division_1(t *testing.T) {
    if i, e := Division(6, 2); i != 3 || e != nil { //try a unit test on function
        t.Error("除法函数测试没通过") // 如果不是如预期的那么就报错
    } else {
        t.Log("第一个测试通过了") //记录一些你期望记录的信息
    }
}

func Test_Division_2(t *testing.T) {
    t.Error("就是不通过")
}
```

我们在项目目录下面执行go test,就会显示如下信息：

```bash
--- FAIL: Test_Division_2 (0.00 seconds)
    gotest_test.go:16: 就是不通过
FAIL
exit status 1
FAIL    gotest  0.013s
```

从这个结果显示测试没有通过，因为在第二个测试函数中我们写死了测试不通过的代码t.Error，那么我们的第一个函数执行的情况怎么样呢？默认情况下执行go test是不会显示测试通过的信息的，我们需要带上参数`go test -v`，这样就会显示如下信息：

```
=== RUN Test_Division_1
--- PASS: Test_Division_1 (0.00 seconds)
    gotest_test.go:11: 第一个测试通过了
=== RUN Test_Division_2
--- FAIL: Test_Division_2 (0.00 seconds)
    gotest_test.go:16: 就是不通过
FAIL
exit status 1
FAIL    gotest  0.012s
```


## 代码覆盖率

```go
go test -cover -v
```

通过在每个代码块中插入布尔类型的变量，统计代码是否被执行。


**可视化**

```go
go test -coverprofile=c.out
go tool cover -html=c.out
```

使用 `-covermode=count` 在每行代码中插入计数器而不是布尔变量。


## 指定超时时间

```shell
go test -v -run=XXX -timeout=10m
```


# 压力测试

压力测试用来检测函数(方法）的性能，和编写单元功能测试的方法类似,此处不再赘述，但需要注意以下几点：

- 压力测试用例必须遵循如下格式，其中XXX可以是任意字母数字的组合，但是首字母不能是小写字母

  `func BenchmarkXXX(b *testing.B) { ... }`

- go test不会默认执行压力测试的函数，如果要执行压力测试需要带上参数`-test.bench`，语法:`-test.bench="test_name_regex"`,例如`go test -test.bench=".*"`表示测试全部的压力测试函数
- `-bench`也可以, bench可以使用`|`(`-bench='A|bcd|EFG'`, `-bench=Benchmark(A|B)`)

- 在压力测试用例中,请记得在循环体内使用testing.B.N,以使测试可以正常的运行

- 文件名也必须以_test.go结尾

```go
package gotest

import (
    "testing"
)

func Benchmark_Division(b *testing.B) {
    for i := 0; i < b.N; i++ { //use b.N for looping
        Division(4, 5)
    }
}

func Benchmark_TimeConsumingFunction(b *testing.B) {
    b.StopTimer() //调用该函数停止压力测试的时间计数

    //做一些初始化的工作,例如读取文件数据,数据库连接之类的,
    //这样这些时间不影响我们测试函数本身的性能

    b.StartTimer() //重新开始时间
    for i := 0; i < b.N; i++ {
        Division(4, 5)
    }
}
```

我们执行命令`go test -file webbench_test.go -test.bench=".*"`，可以看到如下结果：

```
PASS
Benchmark_Division  500000000            7.76 ns/op
Benchmark_TimeConsumingFunction 500000000            7.80 ns/op
ok      gotest  9.364s  
```

上面的结果显示我们没有执行任何TestXXX的单元测试函数，显示的结果只执行了压力测试函数，第一条显示了 `Benchmark_Division` 执行了500000000次，每次的执行平均时间是7.76纳秒，第二条显示了 `Benchmark_TimeConsumingFunction` 执行了500000000，每次的平均执行时间是7.80纳秒。最后一条显示总共的执行时间。


```
go test -v -run=XXX -bench=函数名正则 包路径
```

## 记录cpu占用

```
go test -v -run=XXX -bench=函数名正则 -cpuprofile=cpu.out sms
```

会在当前目录下生成一个以".test"结尾的文件和一个cpu.out,查看的话使用:

```
go tool pprof ./XX.test cpu.out
```

---------------

文本输出：

```go
go tool pprof -text -nodecount=10 ./XX.test cpu.log
```

`-text` 标志参数用于指定输出格式, 在这里每行是一个函数, 根据使用CPU的时间来排
序

`-nodecount=10` 标志参数限制了只输出前10行的结果

## 记录内存分配

```go
添加 -benchmem
```



## 并发测试

```go
b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			// some code
		}
	})
```


# 例子用例

- 文件名也是以 `_test` 结尾
- 使用fmt包输出，使用注释 `// Output: ` 来验证输出是否正确。
- Example[FunctionName] 针对FunctionName的例子，会显示在FunctionName的下，如果没有FunctionName则会显示在Overview下。


```go
func ExampleInts() {
	s := []int{5, 2, 6, 3, 1, 4} // unsorted
	sort.Ints(s)
	fmt.Println(s)
	// Output: [1 2 3 4 5 6]
}
```


# 测试脚本

## 除vendor外的测试

```bash
IGNORE_PKGS="(vendor)"
TEST_PKGS=`find . -name \*_test.go | while read a; do dirname $a; done | sort | uniq | egrep -v "$IGNORE_PKGS" | sed "s|\./||g"`
```

说明：
* `find . -name \*_test.go` 找到所有的测试文件。
* `dirname` 获取全路径中的目录部分，比如 `a/b/c.txt` 获取的是 `a/b`。
* `sed "s|\./||g` 去掉 `./`

最后的结果就是列举出自己写的 `_test.go` 文件。


# 测试技巧


## 子测试

```go
import "testing"

func TestAdd(t *testing.T) {
	a := 1

	t.Run("+1", func(t *testing.T) {
		if a+1 != 2 {
			t.Fatal("fail!")
		}
	})
	t.Run("+2", func(t *testing.T) {
		if a+2 != 3 {
			t.Fatal("fail!")
		}
	})
}
```

* 每个 `Run` 都是相互独立的, 可能会在多个 goroutine 里执行.
* 外层测试会在所有子测试都执行完毕才会 return.
* 单独执行其中一个子测试: `go test -v -run=TestAdd/1`(使用`+1`报错)


## 表格驱动

```go
func TestAdd(t *testing.T) {
    cases := []struct{ A, B, Expected int }{
        { 1, 1, 2 },
        { 1, -1, 0 },
        { 1, 0, 1 },
        { 0, 0, 0 },
    }
    for _, tc := range cases {
        t.Run(fmt.Sprintf("%d + %d", tc.A, tc.B), func(t *testing.T) {
            actual := tc.A + tc.B
            if actual != expected { t.Fatal("failure") }
        })
    }
}
```

给每个用例添加一个名称:

```go
func TestAdd(t *testing.T) {
    cases := []struct{
        Name string
        A, B, Expected int
    }{
      	{"foo", 1, 1, 2 },
        {"bar", 1, -1, 0 },
    }
    for k, tc := range cases {
        t.Run(tc.Name, func(t *testing.T) {
            ...
        })
    }
}
```


## 测试用例的当前目录

```go
func TestAdd(t *testing.T) {
    data := filepath.Join("test-fixtures", "add_data.json")
    // ... Do something with data                                                                        
}
```

* 当前目录就是测试文件所在路径.
* 可以将一些测试数据(比如配置等)放在 `test-fixtures` 目录下, 测试用例中使用相对路径来获取.


## flag

```go
var update = flag.Bool("update", false, "update file")

func TestAdd(t *testing.T) {
	t.Log(*update)
}
```

可以使用下面的语句测试:

```shell
go test -v
go test -v -update
go test -v -update=true
```


## Don’t export concurrency primitives

假设有一个库，提供一个读取消息的接口，并将 channel 暴露给调用者:

```go
type Reader struct {...}
func (r *Reader) ReadChan() <-chan Msg {...}
```

现在一个使用者想实现一个测试用例:

```go
func TestConsumer(t testing.T) {
    cons := &Consumer{
        r: libqueue.NewReader(),
    }
    for msg := range cons.r.ReadChan() {
        // Test thing.
    }
}
```

使用者决定使用依赖注入的方式，使用他们自己的实现来替换 Reader:

```go
func TestConsumer(t testing.T, q queueIface) {
    cons := &Consumer{
        r: q,
    }
    for msg := range cons.r.ReadChan() {
        // Test thing.
    }
}
```

但是怎么处理错误呢？或许你会这样做:

```go
func TestConsumer(t testing.T, q queueIface) {
    cons := &Consumer{
        r: q,
    }
    for {
        select {
        case msg := <-cons.r.ReadChan():
            // Test thing.
        case err := <-cons.r.ErrChan():
            // What caused this again?
        }
    }
}
```

现在，我们应该怎么模拟事件来保证和 Reader 库的表现一致呢？如果库只是提供一个简单的同步API, 那么测试时就简单多了:

```go
func TestConsumer(t testing.T, q queueIface) {
    cons := &Consumer{
        r: q,
    }
    msg, err := cons.r.ReadMsg()
    // handle err, test thing
}
```

记住，不用轻易暴露并发原语给调用者，暴露容易，想要删除或者修改就难了。另外，不要忘记在结构体或库的文档中指明是否是并发安全的。

最后，如果有必要暴露 channel, 可以通过一个访问器，并将 channel 设置为只读(`<-chan`)或只写(`chan<-`)。



## 使用 net/http/httptest

httptest 可以让你在不启动服务或绑定端口的情况下测试你的 `http.Handler` 代码。下面实现了两种方式来测试你的 `ServeHTTP()`:

```go
func TestServe(t *testing.T) {
    // The method to use if you want to practice typing
    s := &http.Server{
        Handler: http.HandlerFunc(ServeHTTP),
    }
    // Pick port automatically for parallel tests and to avoid conflicts
    l, err := net.Listen("tcp", ":0")
    if err != nil {
        t.Fatal(err)
    }
    defer l.Close()
    go s.Serve(l)

    res, err := http.Get("http://" + l.Addr().String() + "/?sloths=arecool")
    if err != nil {
        log.Fatal(err)
    }
    greeting, err := ioutil.ReadAll(res.Body)
    res.Body.Close()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(greeting))
}

func TestServeMemory(t *testing.T) {
    // Less verbose and more flexible way
    req := httptest.NewRequest("GET", "http://example.com/?sloths=arecool", nil)
    w := httptest.NewRecorder()

    ServeHTTP(w, req)
    greeting, err := ioutil.ReadAll(w.Body)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(greeting))
}
```

httptest 还可以启动一个临时服务，监听随机端口: `httptest.NewServer()`。



## Use a separate test package

比如当前包名为 foo, 一般测试文件 `foo_test.go` 中会声明 `package foo`，这样测试文件能访问到 foo 包中所有的字段和函数。

也可以将 `foo_test.go` 中的包声明改成其他名称，比如 `package foo_test`，这样测试 foo 包时就需要导入它。这样做的好处是能从使用者的角度来测试包。难以使用的包同样难以测试。

这样做还有一个好处是避免循环依赖。

比如 `net/url` 包实现了解析 url 的功能，`net/http` 导入了 `net/url` 包，而 `net/url` 包在测试时需要导入 `net/http` 包，这样就会形成循环引用。而如果测试文件中的包名不是 `url` 就可以同时导入 `url` 和 `http` 来测试，以避免循环引用。




# 参考
* [Advanced Testing in Go, Mitchell Hashimoto](https://about.sourcegraph.com/go/advanced-testing-in-go-mitchell-hashimoto)
- [5 Advanced Testing Techniques in Go](https://segment.com/blog/5-advanced-testing-techniques-in-go/)
