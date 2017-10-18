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
    - [gcflags](#gcflags)
- [go list](#go-list)
- [go tool objdump](#go-tool-objdump)

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

## gcflags

关闭编译器代码优化

```go
go build -gcflags "-N" -o test test.go
```

关闭函数内联

```go
go build -gcflags "-l" -o test test.go
```

同时制定

```go
go build -gcflags "-N -l" -o test test.go
```

查看编译优化信息

```go
go build -gcflags "-m" test.go
```



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

