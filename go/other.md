<!-- TOC -->

- [go的一般结构和执行顺序](#go的一般结构和执行顺序)
- [init()](#init)
- [全局函数](#全局函数)
    - [close](#close)
    - [len、cap](#lencap)
    - [new、make](#newmake)
    - [copy、append](#copyappend)
        - [copy(dst, src[]Type) int](#copydst-srctype-int)
    - [panic、recover](#panicrecover)
    - [print、println](#printprintln)
    - [complex、real imag](#complexreal-imag)
- [优化](#优化)

<!-- /TOC -->


# go的一般结构和执行顺序
下面的程序可以被顺利编译但什么都做不了，不过这很好地展示了一个 Go 程序的首选结构。这种结构并没有被强制要求，编译器也不关心 main 函数在前还是变量的声明在前，但使用统一的结构能够在从上至下阅读 Go 代码时有更好的体验。总体思路如下：

- 在完成包的 import 之后，开始对常量、变量和类型的定义或声明。
- 如果存在 init 函数的话，则对该函数进行定义（这是一个特殊的函数，每个含有该函数的包都会首先执行这个函数）。
- 如果当前包是 main 包，则定义 main 函数。
- 然后定义其余的函数，首先是类型的方法，接着是按照 main 函数中先后调用的顺序来定义相关函数，如果有很多函数，则可以按照字母顺序来进行排序。

```go
package main

import "fmt"

import (
	"fmt"
)

const c = "C"

var v int = 5

type T struct{}

func init() {
	// initialization of package
}

func main() {
	var a int
	Func1()
	// ...
	fmt.Println(a)
}

func (t T) Method1() {
	//...
}

func Func1() {
	// exported function Func1
	//...
}
```


Go 程序的执行（程序启动）顺序如下：
- 按顺序导入所有被 main 包引用的其它包，然后在每个包中执行如下流程：
- 如果该包又导入了其它的包，则从第一步开始递归执行，但是每个包只会被导入一次。
- 然后以相反的顺序在每个包中初始化常量和变量，如果该包含有 init 函数的话，则调用该函数。在完成这一切之后，main 也执行同样的过程，最后调用 main 函数开始执行程序。


# init()
* 每个源文件可以定义自己的不带参数的（niladic）init函数，来设置它所需的状态。（**实际上每个文件可以有多个init函数**。）
* init是在程序包中所有变量声明都被初始化，以及所有被导入的程序包中的变量初始化之后才被调用。
* 除了用于无法通过声明来表示的初始化以外，init函数的一个常用法是在真正执行之前进行验证或者修复程序状态的正确性。

```go
func init() {
	if user == "" {
		log.Fatal("$USER not set")
	}
	if home == "" {
		home = "/home/" + user
	}
	if gopath == "" {
		gopath = home + "/go"
	}
	// gopath may be overridden by --gopath flag on command line.
	flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

  ​

# 全局函数
* 全局函数的定义可在builtin包中查看

## close
* 用于管道通信 `close(ch)`

## len、cap 
* len用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；
* cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map）

## new、make
* new 和 make 均是用于分配内存：
* new 用于值类型和用户定义的类型，如自定义结构，make 用户内置引用类型（切片、map 和管道）。
* 它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。
* new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针。它也可以被用于基本类型：`v := new(int)`。
* make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作。
* new() 是一个函数，不要忘记它的括号

## copy、append
* 用于复制和连接切片 `copy(dst, src)`, `s = append(s, s2...)`

### copy(dst, src[]Type) int

将src复制到dst，返回复制的Type类型的元素的个数

```go
b := make([]byte, 5)
str := []string{"12", "2", "3"}
bp := copy(b, str[0])    // 一个string类型自动编译为[]byte类型？
fmt.Println(bp, b)       // 2 [49 50 0 0 0]
bp = copy(b, str[1])     // 还是复制到b的开始位置
fmt.Println(bp, b)       // 1 [50 50 0 0 0]
bp = copy(b[2:], str[2]) // 将3复制到2位置
fmt.Println(bp, b)       // 1 [50 50 51 0 0]
```

从strings.go的Join()截取的代码：

```go
b := make([]byte, n)
bp := copy(b, a[0]) // 将a[0]当做[]byte复制到b开始的位置，返回a[0]中byte的个数
for _, s := range a[1:] {
	bp += copy(b[bp:], sep)	
	bp += copy(b[bp:], s)}
```

## panic、recover

两者均用于错误处理机制, recover()的调用仅当它在defer函数中被直接调用时才有效.

下面的代码中recover不是在defer中直接调用的，所以它收不到错误信息。

```go
func doRecover() {
	fmt.Println("recovered =>", recover()) //prints: recovered => <nil>
}

func main() {
	defer func() {
		doRecover() //panic is not recovered
	}()
	panic("not good")
}
```

## print、println

底层打印函数，在部署环境中建议使用 fmt 包

## complex、real imag

用于创建和操作复数


# 优化
* [官方调试指南](https://tip.golang.org/doc/diagnostics.html)
