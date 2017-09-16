<!-- TOC -->

- [包](#包)
    - [标准库](#标准库)
    - [编译](#编译)
    - [import](#import)
        - [import多个包：](#import多个包)
        - [import使用说明](#import使用说明)
    - [权威导入路径(import paths)](#权威导入路径import-paths)
    - [初始化](#初始化)

<!-- /TOC -->


# 包

一个应用程序可以包含不同的包，而且即使你只使用 main 包也不必把所有的代码都写在一个巨大的文件里：你可以用一些较小的文件，并且在每个文件非注释的第一行都使用 package main 来指明这些文件都属于 main 包。

如果你打算编译包名不是为 main 的源文件，如 pack1，编译后产生的对象文件将会是 pack1.a 而不是可执行程序。

另外要注意的是，所有的包名都应该使用小写字母。

每个目录都只包含一个包。

如果对一个包进行更改或重新编译，所有引用了这个包的客户端程序都必须全部重新编译。

## 标准库

一般情况下，标准包会存放在 `$GOROOT/pkg/$GOOS_$GOARCH/` 目录下。如：`pkg\windows_386，pkg\linux_amd64`。

## 编译

Go 中的包模型采用了显式依赖关系的机制来达到快速编译的目的，编译器会从后缀名为 .o 的对象文件（需要且只需要这个文件）中提取传递依赖类型的信息。

如果 A.go 依赖 B.go，而 B.go 又依赖 C.go：

编译 C.go, B.go, 然后是 A.go.

为了编译 A.go, 编译器读取的是 B.o 而不是 C.o.

这种机制对于编译大型的项目时可以显著地提升编译速度。


## import

同一包内的go文件中的package要一样

import的时候从src下开始到go文件的父目录即可

相同package的文件类似于处于同一个类中，函数是类方法，struct是内部类，同一个包的文件相互访问不需要import

### import多个包：

```go
import "fmt"
import "os"
```

或：

```go
import "fmt"; import "os"
```

或：

```go
import (
    "fmt"
    "os"
)
```

### import使用说明

Go的import支持如下两种方式来加载自己写的模块

1. 相对路径
   
	```go
	import "./model" //当前文件同一目录的model目录，但是不建议这种方式来import
	```

2. 绝对路径
  
  ```go
  import “shorturl/model” //加载gopath/src/shorturl/model模块
  ```
上面展示了一些import常用的几种方式，但是还有一些特殊的import，让很多新手很费解，下面我们来一一讲解一下到底是怎么一回事


**点操作**

我们有时候会看到如下的方式导入包

```go
import(
	. "fmt"
)
```

这个点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名，也就是前面你调用的 `fmt.Println("hello world")` 可以省略的写成 `Println("hello world")`


**别名操作**

别名操作顾名思义我们可以把包命名成另一个我们用起来容易记忆的名字

```go
import(
	f "fmt"
)
```

别名操作的话调用包函数时前缀变成了我们的前缀，即 `f.Println("hello world")`


**_操作**

这个操作经常是让很多人费解的一个操作符，请看下面这个import

```go
import (
	"database/sql"
	_ "github.com/ziutek/mymysql/godrv"
)
```

_操作其实是引入该包，而不直接使用包里面的函数，而是调用了该包里面的init函数，要理解这个问题，需要看下面这个图，理解包是怎么按照顺序加载的：

程序的初始化和执行都起始于main包。如果main包还导入了其它的包，那么就会在编译时将它们依次导入。有时一个包会被多个包同时导入，那么它只会被导入一次（例如很多包可能都会用到fmt包，但它只会被导入一次，因为没有必要导入多次）。当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行init函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开始对main包中的包级常量和变量进行初始化，然后执行main包中的init函数（如果存在的话），最后执行main函数。下图详细地解释了整个执行过程：

![](pic/package01.png)

通过上面的介绍我们了解了import的时候其实是执行了该包里面的init函数，初始化了里面的变量，_操作只是说该包引入了，我只初始化里面的init函数和一些变量，但是往往这些init函数里面是注册自己包里面的引擎，让外部可以方便的使用，就很多实现database/sql的引起，在init函数里面都是调用了sql.Register(name string, driver driver.Driver)注册自己，然后外部就可以使用了。



## 权威导入路径(import paths)

我们经常使用托管在公共代码托管服务中的代码，诸如github.com，这意味着包导入路径包含托管服务名，比如github.com/rsc /pdf。一些场景下为了不破坏用户代码，我们用rsc.io/pdf，屏蔽底层具体哪家托管服务，比如rso.io/pdf的背后可能是 github.com也可能是bitbucket。但这样会引入一个问题，那就是不经意间我们为一个包生成了两个合法的导入路径。如果一个程序中 使用了这两个合法路径，一旦某个路径没有被识别出有更新，或者将包迁移到另外一个不同的托管公共服务下去时，使用旧导入路径包的程序就会报错。

Go 1.4引入一个包字句的注释，用于标识这个包的权威导入路径。如果使用的导入的路径不是权威路径，go命令会拒绝编译。语法很简单：

```go
package pdf // import "rsc.io/pdf"
```

如果pdf包使用了权威导入路径注释，那么那些尝试使用github.com/rsc/pdf导入路径的程序将会被go编译器拒绝编译。

这个权威导入路径检查是在编译期进行的，而不是下载阶段。

我们举个例子：

我们的包foo以前是放在github.com/bigwhite/foo下面的，后来主托管站换成了tonybai.com/foo，最新的 foo包的代码：

```go
package foo // import "tonybai.com/foo"

import "fmt"

func Echo(a string) {
        fmt.Println("Foo:, a)
}
```

某个应用通过旧路径github.com/bigwhite/foo导入了该包：

```go
//testcanonicalimportpath.go
package main

import "github.com/bigwhite/foo"

func main() {
        foo.Echo("Hello!")
}
```

我们编译该go文件，得到以下结果：

```
code in directory /home/tonybai/Test/Go/src/github.com/bigwhite/foo expects import "tonybai.com/foo"
```

## 初始化

包的初始化首先是解决包级变量的依赖顺序, 然后按照包级变量声明出现的顺序依次初始化:

```go
var a = b + c // a第三个初始化,为3 
var b = f()    // b第二个初始化,为2,通过调用f (依赖c) 
var c = 1      // c第一个初始化,为1

func f() int { return c + 1 }
```

如果包中含有多个.go 文件, 它们按照发给编译器的顺序进行初始化, Go的构建工具首先将.go 文件根据文件名排序, 然后依次调用编译器编译.