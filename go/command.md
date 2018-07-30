<!-- TOC -->

- [go generate](#go-generate)
    - [一个简单的例子:mkdir](#一个简单的例子mkdir)
- [在程序中执行命令](#在程序中执行命令)
- [go test](#go-test)
- [go fmt](#go-fmt)
- [goimports](#goimports)
- [goreturns](#goreturns)
- [golint](#golint)
- [go install](#go-install)
- [go build](#go-build)
    - [交叉编译](#交叉编译)
    - [ldflags](#ldflags)
        - [编译时加版本号](#编译时加版本号)
    - [gcflags](#gcflags)
    - [条件编译](#条件编译)
- [go list](#go-list)
- [go tool objdump](#go-tool-objdump)
- [go clean](#go-clean)
- [go tool pprof](#go-tool-pprof)
- [go tool trace](#go-tool-trace)
    - [生成 trace 数据的方式](#生成-trace-数据的方式)
    - [显示](#显示)
- [-race 参数检查数据竞争](#-race-参数检查数据竞争)
- [Makefile](#makefile)

<!-- /TOC -->


# go generate

## 一个简单的例子:mkdir

```go
package main

//go:generate mkdir ~/test

import (
	"fmt"
)

func main() {
	fmt.Println("go generate test finished.")
}
```

在命令行中执行

```
go generate -v -n -x /代码路径/main.go
```

输出：

```
go/code/src/main.go
mkdir ~/test
```

test文件夹也生成了。



# 在程序中执行命令 #

```go
	cmd := exec.Command("hugo", "-t=hyde")
	cmd.Dir = "D:/work/hugo/mysite"
	out, err := cmd.CombinedOutput()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(string(out))
```

执行 sh 文件:

```go
exec.Command("/bin/sh", "/abc.sh")
```


# go test #

```go
go test internal/config -run=TestKindsString
```

* 在src目录的父目录下
* config是目录，下面有一些 `*_test.go` 文件
* -run 可以指定执行哪个测试函数。


# go fmt
* 格式化整个项目并将结果写入文件： `gofmt -l -d -w src/abc`
* `gofmt –w program.go` 会格式化该源文件的代码然后将格式化后的代码覆盖原始内容（如果不加参数 `-w` 则只会打印格式化后的结果而不重写文件）；
* `gofmt -w *.go` 会格式化并重写所有 Go 源文件；
* `gofmt map1` 会格式化并重写 map1 目录及其子目录下的所有 Go 源文件。
* `gofmt` 也可以通过在参数 `-r` 后面加入用双引号括起来的替换规则实现代码的简单重构，规则的格式：`<原始内容> -> <替换内容>`。

示例:
* 将源文件中没有意义的括号去掉: `gofmt -r "(a) -> a" –w *.go`
* 将源文件中多余的 len(a) 去掉: `gofmt -r "a[n:len(a)] -> a[n:]" –w *.go`
* 将源文件中符合条件的函数的参数调换位置: `gofmt –r 'A.Func1(a,b) -> A.Func2(b,a)' –w *.go`

# goimports

```
go get golang.org/x/tools/cmd/goimports
```

* 和 gofmt 相结合的.
* 自动删除无用的 import 和添加未导入的包.


# goreturns

```
go get sourcegraph.com/sqs/goreturns
```

* 函数返回时, 如果参数个数不匹配, 会自动添加缺失参数的零值.
* 做完自己的事情后会调用 goimports.


# golint

golint不能对目录的子目录执行，对所有目录执行的shell方法：

```bash
find src/push-sms -name "*.go" -exec golint {} \;
```


# go install

* 安装所有程序(在src下执行): `go install -v ./...`



# go build

* `go build`: 编译当前目录对应的代码包.
* `go build my_pkg`: 编译 `my_pkg` 包.
* `go build aa/a.go aa/b.go`: 编译多个文件, 必须在同一目录下.




## 交叉编译

```shell
GOOS=linux GOARCH=amd64 go build
GOOS=windows GOARCH=amd64 go build
```

不支持cgo.

## ldflags

采用：`go build -ldflags "-s -w"` 这种方式编译。

解释一下参数的意思：

- -ldflags： 表示将后面的参数传给连接器（5/6/8l）
- -s：去掉符号信息
- -w：去掉DWARF调试信息
  注意：

`-s` 去掉符号表（这样panic时，stack trace就没有任何文件名/行号信息了，这等价于普通C/C+=程序被strip的效果）

`-w` 去掉DWARF调试信息，得到的程序就不能用gdb调试了

两个可以分开使用


参数列表可以使用下面的语句查看：

```go
go tool compile -help
```

### 编译时加版本号

```go
var Version = "unknown" // 使用小写 version 也可以。

func main() {
	fmt.Println("version:", Version)
}
```

编译时：

```bash
go run -ldflags "-X main.Version=v1.0.0" main.go

# 设置多个变量
go run -ldflags "-X main.Version1=v1.0.0 -X main.Version2=v2.0.0" main.go

# 使用 install
go install -ldflags "-X main.Version1=v1.0.0 -X main.Version2=v2.0.0" -v ./...

# 使用其他包
go install -ldflags "-X github.com/aaa/test.Version=v1.0.0" -v ./...
```

`-X` 的文档可以在 https://golang.org/cmd/link/ 中查到。



## gcflags

关闭编译器代码优化

```bash
go build -gcflags "-N" -o test test.go
```

关闭函数内联

```bash
go build -gcflags "-l" -o test test.go
```

同时制定

```bash
go build -gcflags "-N -l" -o test test.go
```

查看编译优化信息

```bash
# 内存逃逸分析 
go build -gcflags "-m" test.go
go build -gcflags '-m -l' test.go    # 关闭函数内联
go build -gcflags '-m -m -l' test.go # -m 越多分析的越细
```


## 条件编译

* https://golang.org/pkg/go/build/#hdr-Build_Constraints

```go
// +build linux,386 darwin,!cgo
```

`,` 对应的是 AND, 空格对应的是 OR, 所以上面对应的是 `(linux AND 386) OR (darwin AND (NOT cgo))`.

`+build` 放到文件顶部, 后面需要跟一行空行.

多行 build 之间使用 AND 联结, 比如:

```go
// +build linux darwin
// +build 386
```

对应的意思是 `(linux OR darwin) AND 386`.

build 后可以跟的词组:

- the target operating system, as spelled by runtime.GOOS
- the target architecture, as spelled by runtime.GOARCH
- the compiler being used, either "gc" or "gccgo"
- "cgo", if ctxt.CgoEnabled is true
- "go1.1", from Go version 1.1 onward
- "go1.2", from Go version 1.2 onward
- "go1.3", from Go version 1.3 onward
- "go1.4", from Go version 1.4 onward
- "go1.5", from Go version 1.5 onward
- "go1.6", from Go version 1.6 onward
- "go1.7", from Go version 1.7 onward
- "go1.8", from Go version 1.8 onward
- "go1.9", from Go version 1.9 onward
- "go1.10", from Go version 1.10 onward
- any additional words listed in ctxt.BuildTags


通过文件名来决定是否要编译:

```go
*_GOOS
*_GOARCH
*_GOOS_GOARCH
```

比如: `source_windows_amd64.go`.

使一个文件不被编译: `// +build ignore`, 其他不满足的词都可以, 只是 ignore 更适合.


build 后可以跟自定义的标签, 比如 `// +build mytag`, `// +build !mytag`, build 时使用 `go build -tags 'mytag'` 分开编译, 多个 tag 使用空格分隔.






# go list

查看项目引用包

```
go list -json
```

查看某个包引用了哪些包:

```shell
go list -f '{{ .Imports }}' github.com/a/b
```

查看标准库的依赖关系: `go list -json std`




# go tool objdump

查看生成的汇编代码

```go
package main

import "fmt"

func main() {
	fmt.Println(*test())
}

func test() *int {
	x := new(int)
	*x = 0xAABB
	return x
}
```

```bash
go tool objdump -s "main\.test" test
```

输出：

```bash
TEXT main.test(SB) /Users/zhangyuchen/tmp/test.go
	test.go:9	0x20f0	65488b0c25a0080000	GS MOVQ GS:0x8a0, CX
	test.go:9	0x20f9	483b6110		CMPQ 0x10(CX), SP
	test.go:9	0x20fd	7639			JBE 0x2138
	test.go:9	0x20ff	4883ec18		SUBQ $0x18, SP
	test.go:9	0x2103	48896c2410		MOVQ BP, 0x10(SP)
	test.go:9	0x2108	488d6c2410		LEAQ 0x10(SP), BP
	test.go:10	0x210d	488d054c700800		LEAQ 0x8704c(IP), AX
	test.go:10	0x2114	48890424		MOVQ AX, 0(SP)
	test.go:10	0x2118	e8c3c90000		CALL runtime.newobject(SB)
	test.go:10	0x211d	488b442408		MOVQ 0x8(SP), AX
	test.go:11	0x2122	48c700bbaa0000		MOVQ $runtime.mapaccess2_fast64+459(SB), 0(AX)
	test.go:12	0x2129	4889442420		MOVQ AX, 0x20(SP)
	test.go:12	0x212e	488b6c2410		MOVQ 0x10(SP), BP
	test.go:12	0x2133	4883c418		ADDQ $0x18, SP
	test.go:12	0x2137	c3			RET
	test.go:9	0x2138	e803c90400		CALL runtime.morestack_noctxt(SB)
	test.go:9	0x213d	ebb1			JMP main.test(SB)
	:-1		0x213f	cc			INT $0x3
```


# go clean

[参考](http://wiki.jikexueyuan.com/project/go-command-tutorial/0.4.html)

执行 `go clean` 命令会删除掉执行其它命令时产生的一些文件和目录，包括：

* 在使用go build命令时在当前代码包下生成的与包名同名或者与Go源码文件同名的可执行文件。在Windows下，则是与包名同名或者Go源码文件同名且带有“.exe”后缀的文件。
* 在执行 `go test` 命令并加入 `-c` 标记时在当前代码包下生成的以包名加 `.test` 后缀为名的文件。在Windows下，则是以包名加 `.test.exe` 后缀为名的文件。
* 如果执行 `go clean` 命令时带有标记 `-i`，则会同时删除安装当前代码包时所产生的结果文件。如果当前代码包中只包含库源码文件，则结果文件指的就是在工作区的pkg目录的相应目录下的归档文件。如果当前代码包中只包含一个命令源码文件，则结果文件指的就是在工作区的bin目录下的可执行文件。
* 还有一些目录和文件是在编译Go或C源码文件时留在相应目录中的。包括：`_obj`和`_test`目录，名称为`_testmain.go`、`test.out`、`build.out`或`a.out`的文件，名称以`.5`、`.6`、`.8`、`.a`、`.o`或`.so`为后缀的文件。这些目录和文件是在执行 `go build` 命令时生成在临时目录中的。临时目录的名称以go-build为前缀。
* 如果执行 `go clean` 命令时带有标记 `-r`，则还包括当前代码包的所有依赖包的上述目录和文件。
* `-n` 标记会打印使用到的系统命令, 但不会真正执行它们.
* `-x` 标记会打印使用到的系统命令, 但会执行它们.


# go tool pprof


# go tool trace

* [深入浅出 Go trace](https://mp.weixin.qq.com/s/I9xSMxy32cALSNQAN8wlnQ)

## 生成 trace 数据的方式

在程序中使用下面的代码将数据输出到标准错误输出中：

```go
import (
    "os"
    "runtime/trace"
)

func main() {
    trace.Start(os.Stderr)
    defer trace.Stop()
    ch := make(chan int)

    go func() {
        ch <- 42
    }()
    <-ch
}
```

之后使用 `go run main.go 2> trace.out` 输出到 trace.out 文件中，再使用 `go tool trace trace.out` 读取 trace。（会打开浏览器显示，默认会随机分配端口，也可以指定：`go tool trace --http=':8080' trace.out`）


## 显示

**View Trace**

* timeline 可以使用 WASD 来左右移动和放大缩小。
* heap 显示内存分配和回收。
* goroutines 显示正在运行的协程数和可运行的协程数。
* threads 显示正在使用的 OS 线程和被 syscall 阻塞的。
* Virtual Processors 虚拟处理器，数量由 GOMAXPROCS 控制。




# -race 参数检查数据竞争

```shell
go run -race *.go
go build -race *.go
go test -v -race
```




# Makefile

摘抄自: https://ops.tips/blog/minimal-golang-makefile/:

```makefile
# I usually keep a `VERSION` file in the root so that anyone
# can clearly check what's the VERSION of `master` or any
# branch at any time by checking the `VERSION` in that git
# revision
VERSION         :=      $(shell cat ./VERSION)
IMAGE_NAME      :=      cirocosta/l7

# As a call to `make` without any arguments leads to the execution
# of the first target found I really prefer to make sure that this
# first one is a non-destructive one that does the most simple
# desired installation. It's very common to people set it as `all`
# but it could be anything like `a`.
all: install

# Install just performs a normal `go install` which builds the source
# files from the package at `./` (I like to keep a `main.go` in the root
# that imports other subpackages). As I always commit `vendor` to `git`
# a `go install` will typically always work - except if there's an OS
# limitation in the build flags (e.g, a linux-only project).
install:
	go install -v

# keeping `./main.go` with just a `cli` and `./lib/*.go` with actual
# logic, `tests` usually reside under `./lib` (or some other subdirectories).
# Here we could do something like `find . -name "*" -type d -exec ...` but IMO
# that's unnecessary. Just `cd`ing to what matters to you is fine - no need to
# handle the case of directories that you don't want to execute a command.
test:
	cd ./lib && go test -v

# Just like `test`, formatting what matters. As `main.go` is in the root,
# `go fmt` the root package. Then just `cd` to what matters to you (`vendor`
# doesn't matter).
# fmt:
	go fmt
	cd ./lib && go fmt


# This target is only useful if you plan to also create a Docker image at
# the end. I have a separate `gist` with a sample Dockerfile tailored for
# golang that you can check out at <TODO>.
# I really like publishing a Docker image together with the GitHub release
# because Docker makes it very simple to someone run your binary without
# having to worry about the retrieval of the binary and execution of it
# - docker already provides the necessary boundaries.
image:
	docker build -t cirocosta/l7 .


# This is pretty much an optional thing that I tend to always include.
# Goreleaser is a tool that allows anyone to integrate a binary releasing
# process to their pipelines. Here in this target With just a simple
# `make release` you can have a `tag` created in GitHub with multiple
# builds if you wish.
# See more at `gorelease` github repo.
release:
	git tag -a $(VERSION) -m "Release" || true
	git push origin $(VERSION)
	goreleaser --rm-dist

.PHONY: install test fmt release
```
