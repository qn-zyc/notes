<!-- TOC -->

- [first demo](#first-demo)
- [利用VisitAll自动生成flag信息](#利用visitall自动生成flag信息)
- [命令行参数](#命令行参数)
    - [os.Args](#osargs)
- [os.Args vs flag.Args](#osargs-vs-flagargs)
- [flag set](#flag-set)
    - [lookup获取的值的类型](#lookup获取的值的类型)
    - [多个flag set](#多个flag-set)

<!-- /TOC -->



先查看godoc文档

# first demo

> 习自 <https://github.com/on99/gocyclo/blob/master/gocyclo.go>

```go
package flag_demo

import (
	"flag"
	"fmt"
	"os"
)

const usageDoc = `Calculate cyclomatic complexities of Go functions.
Usage:
		gocyclo [flags] <Go file or directory> ...
		
Flags:
		-over N balabala..
		-top N balabala...
		-avg balabala...

...
`

func usage() {
	fmt.Fprintf(os.Stderr, usageDoc)
	os.Exit(2)
}

var (
	over = flag.Int("over", 0, "show functions with complexity > N only")
	top  = flag.Int("top", -1, "show the top N most complex functions only")
	avg  = flag.Bool("avg", false, "show the average complexity")
)

func MyMain() {
	flag.Usage = usage
	flag.Parse()

	args := flag.Args()
	if len(args) == 0 {
		usage()
	}
	fmt.Println("args: ", args)

	fmt.Println("over: ", *over)
	fmt.Println("top: ", *top)
	fmt.Println("avg: ", *avg)
}
```

使用1：`./src`

输出：

```
Calculate cyclomatic complexities of Go functions.
Usage:
		gocyclo [flags] <Go file or directory> ...
		
Flags:
		-over N balabala..
		-top N balabala...
		-avg balabala...

...
```

使用2: `./src -over=5 -top=9 -avg=true some thing`

输出：

```
args:  [some thing]
over:  5
top:  9
avg:  true
```

# 利用VisitAll自动生成flag信息 

```go
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	usage := GenerateUsageFunc()
	ParseFlag(usage)
	fmt.Println(myvar)
}

var myvar string

func init() {
	flag.StringVar(&myvar, "myarg", "def", "whatever...")
}

func GenerateUsageFunc() func() {
	name := os.Args[0] // 程序路径
	return func() {
		fmt.Fprintf(os.Stderr, "Usage of %s:\n", name)
		fmt.Fprintf(os.Stderr, "\t%s [flags] \n", name)
		fmt.Fprintf(os.Stderr, "Flags:\n")
		// 这里就是自动生成 flag 的信息
		flag.VisitAll(func(fg *flag.Flag) {
			fmt.Fprintf(os.Stderr, "  -%s = %s (%s)\n", fg.Name, fg.DefValue, fg.Usage)
		})
	}
}

func ParseFlag(usageFunc func()) {
	// CommandLine 是 flag 包中某些全局函数的默认实现，其实就是一个 FlagSet
	flag.CommandLine.Init(os.Args[0], flag.ExitOnError)
	flag.CommandLine.SetOutput(os.Stderr)
	flag.CommandLine.Usage = usageFunc
	flag.Parse()
	flag.VisitAll(func(fg *flag.Flag) {
		fmt.Printf("your flag: [%10s]=%10s \n", fg.Name, fg.Value.String())
	})
}
```

输出帮助信息 `test.exe -h`:

```
Usage of D:\go\code\test\bin\test.exe:
        D:\go\code\test\bin\test.exe [flags]
Flags:
  -myarg = def (whatever...)

```

设置正确的参数 `test.exe -myarg=a`

```
your flag: [     myarg]=         a
a
```

设置错误的参数 `test.exe -myerr=a`

```
flag provided but not defined: -myerr
Usage of D:\go\code\test\bin\test.exe:
        D:\go\code\test\bin\test.exe [flags]
Flags:
  -myarg = def (whatever...)

```

# 命令行参数

> <http://golang-china.github.io/gopl-zh/ch1/ch1-02.html>

## os.Args ##

os.Args这个变量是一个字符串(string)的slice

os.Args的第一个元素，即os.Args[0]是命令行执行时的命令本身；其它的元素则是执行该命令时传给这个程序的参数

# os.Args vs flag.Args

```go
func main() {
	flag.Parse()
	fmt.Println(os.Args)
	fmt.Println(flag.NArg(), flag.Args())
}
```

执行命令：

```bash
./test cmd1 cmd2 -arg1=v1 -arg2=v2
```

执行结果：

```
[./test cmd1 cmd2 -arg1=v1 -arg2=v2]
4 [cmd1 cmd2 -arg1=v1 -arg2=v2]
```

`os.Args` 比 `flag.Args` 多了一个运行路径，但 `flag` 需要先 `parse`

如果添加了下面的代码：

```go
func init() {
	flag.String("arg1", "", "")
	flag.String("arg2", "", "")
}
```

再次执行 `./test cmd1 cmd2 -arg1=v1 -arg2=v2` 时输出的就是：

```
[./test -arg1=v1 -arg2=v2 cmd1 cmd2]
2 [cmd1 cmd2]
```

`flag.Args()` 没有返回已经被解析的 `-arg1` 和 `-arg2`。



# flag set

```go
var (
	flagset = flag.NewFlagSet("test", flag.ExitOnError)
	arg1    = flagset.String("arg1", "arg1value", "usage of arg1")
)

func main() {
	flagset.Parse(os.Args[1:])
	fmt.Println(*arg1)
}
```

## lookup获取的值的类型

```go
var flagset = flag.NewFlagSet("hello", flag.ExitOnError)

func init() {
	flagset.Int("a", 1, "usage")
}

func main() {
	flagset.Parse(os.Args[1:])
	vv := flagset.Lookup("a").Value.(flag.Getter).Get().(int)
	fmt.Println(vv)
}
```

## 多个flag set ##

```go
package main

import (
	"flag"
	"fmt"
	"os"
)

var flagset = flag.NewFlagSet("test", flag.ExitOnError)
var flagset2 = flag.NewFlagSet("test2", flag.ExitOnError)

func init() {
	flagset.String("arg1", "value1", "haha")
	flagset2.String("arg1", "value2", "hehe")
}

func main() {
	flagset.Parse(os.Args[1:])
	flagset2.Parse(os.Args[1:])

	f := flagset.Lookup("arg1")
	fmt.Println(f.Value)

	f = flagset2.Lookup("arg1")
	fmt.Println(f.Value)
}
```

usage 只显示第一个flagset的：

```
Usage of test:
  -arg1 string
        haha (default "value1")
```

并且设置了arg1后两个flagset都是可以获取到新值的。

```
-arg1=abc
```

如果两个flagset的变量一个是arg1,一个是arg2，那么设置了其中一个，另一个就会报错，没有定义这个变量，所以，***flagset只能用一个***。

