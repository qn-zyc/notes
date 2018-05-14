<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [测试用例](#测试用例)
	- [代码覆盖率](#代码覆盖率)
- [压力测试](#压力测试)
	- [记录cpu占用](#记录cpu占用)
- [例子用例](#例子用例)
- [测试脚本](#测试脚本)
	- [除vendor外的测试](#除vendor外的测试)
	- [测试覆盖率](#测试覆盖率)
- [测试技巧](#测试技巧)
	- [子测试](#子测试)
	- [表格驱动](#表格驱动)
	- [测试用例的当前目录](#测试用例的当前目录)
	- [flag](#flag)
	- [Don’t export concurrency primitives](#dont-export-concurrency-primitives)
	- [使用 net/http/httptest](#使用-nethttphttptest)
	- [Use a separate test package](#use-a-separate-test-package)
	- [TestMain](#testmain)
- [参考](#参考)

<!-- /TOC -->


# 测试用例

- 文件名必须是 `_test.go` 结尾的，这样在执行 `go test` 的时候才会执行到相应的代码
- 测试包是 `testing`。
- 所有的测试用例函数必须是 `Test` 开头, 比如 `TestAdd(t *testing.T)`, Add 首字母需要大写。
- 测试用例会按照源代码中写的顺序依次执行。
- 函数中通过调用 `testing.T` 的 Error, Errorf, FailNow, Fatal, FatalIf 方法，说明测试不通过，调用 Log 方法用来记录测试的信息。
- 执行指定单测: `go test -v -run=函数名正则 包路径`。
- 执行当前目录下所有的单元测试: `go test -v ./...`。
- 指定超时时间: `go test -v -run=XXX -timeout=10m`。



## 代码覆盖率

- 显示总的覆盖率信息: `go test -v -cover`。
- 输出覆盖率信息到文件并用浏览器显示: `go test -coverprofile=c.out && go tool cover -html=c.out`。
- covermode: `go test -v covermode=atomic`。 通过在代码中插入埋点来实现的。
    - set: 每个语句是否执行到, 默认配置。
    - count: 每个语句执行了多少次。
    - atomic: 类似于 count, 但表示的是并行程序中的精确计数。



# 压力测试

- 压力测试用来检测函数(方法)的性能。
- 函数声明格式: `func BenchmarkXXX(b *testing.B) { ... }`
- go test 默认执行单测, 而不执行压力测试, 需要使用 `go test -v -bench=函数名正则` 来指定。
    - `go test -v -run=xxx -bench=.`: 不执行单测, 执行所有压测。
    - `go test -v -bench='A|bcd|EFG'`: 使用 `|` 表示或。
    - `go test -v -bench=Benchmark(A|B)`
- 文件名也必须以 `_test.go` 结尾。
- cpu 占用: `go test -v -cpuprofile=cpu.out`。
- 内存占用: `go test -v -benchmem`。 使用 `go test -v -memprofile=mem.out` 输出 mem.out 用于 go tool pprof 分析.


## 记录cpu占用

```bash
go test -v -run=XXX -bench=函数名正则 -cpuprofile=cpu.out 包路径或无
```

会在当前目录下生成一个以 `.test` 结尾的文件和一个 cpu.out, 查看的话使用:

```bash
go tool pprof ./XX.test cpu.out
```

---------------

文本输出：

```bash
go tool pprof -text -nodecount=10 ./XX.test cpu.out
```

- text 标志参数用于指定输出格式, 在这里每行是一个函数, 根据使用CPU的时间来排序。
- nodecount=10 标志参数限制了只输出前10行的结果。




# 例子用例

- 文件名也是以 `_test` 结尾
- 使用 fmt 包输出，使用注释 `// Output: ` 来验证输出是否正确。


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


## 测试覆盖率

```bash
#! /bin/bash

set -e

mode="count"
coverprofile="c.out"
for package in $(go list ./... | grep -Ev 'qiniu.com/fusionartisan/qtest'); do
    coverprofile="$(echo $package | tr / -).cover"
    go test -covermode="$mode" -coverprofile="$coverprofile" -coverpkg="$package" "$package"
done
```


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

```go
func TestGroupedParallel(t *testing.T) {
    for _, tc := range tests {
        tc := tc // capture range variable
        t.Run(tc.Name, func(t *testing.T) {
            t.Parallel()
            ...
        })
    }
}
```

- 每个子测试相互之间并行执行。


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


## TestMain

如果想在所有测试执行前后做一些设置和回收工作，可以添加一个 `TestMain` 函数。

`TestMain` 中需要调用 `os.Exit()`，参数是 `m.Run()` 返回的值: `os.Exit(m.Run())`。

因为是使用 `os.Exit()` 退出的，所以 defer 不会被执行。

如果使用了 flag，还需要手动执行 `flag.Parse()`:

```go
func TestMain(m *testing.M) {
	// call flag.Parse() here if TestMain uses flags
	os.Exit(m.Run())
}
```

```go
func TestMain(m *testing.M) {
    fmt.Println("1")
    code := m.Run()
    fmt.Println("2")
    defer func() {
        fmt.Println("3")
    }()
    os.Exit(code)
}

func TestABC(t *testing.T) {
    fmt.Println("4")
}
```

输出示例:

```
1
=== RUN   TestABC
4
--- PASS: TestABC (0.00s)
PASS
2
```



# 参考
- [Advanced Testing in Go, Mitchell Hashimoto](https://about.sourcegraph.com/go/advanced-testing-in-go-mitchell-hashimoto)
- [5 Advanced Testing Techniques in Go](https://segment.com/blog/5-advanced-testing-techniques-in-go/)
- [https://golang.org/pkg/testing/](https://golang.org/pkg/testing/)
- [Go多个pkg的单元测试覆盖率](http://singlecool.com/2017/06/11/golang-test/)
