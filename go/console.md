<!-- TOC -->

- [从控制台读取输入](#从控制台读取输入)
    - [bufio](#bufio)
    - [fmt](#fmt)
- [向错误流输出信息](#向错误流输出信息)
- [清屏](#清屏)
- [优雅退出程序（监听ctrl+c事件）](#优雅退出程序监听ctrlc事件)

<!-- /TOC -->


# 从控制台读取输入

## bufio

```go
import (
	"bufio"
	"os"
)

var r *bufio.Reader = bufio.NewReader(os.Stdin)
rawLine, _, _ := r.ReadLine()
line := string(rawLine)
```

rawline是[]byte类型

应该使用 `ReadString('\n')`，读取的字符串包括 '\n' 字符。

下面是循环读取的例子：

```go
var r *bufio.Reader = bufio.NewReader(os.Stdin)
for {
	fmt.Print("Enter command ->")
	rawLine, _, _ := r.ReadLine()
	line := string(rawLine)

	if line == "q" || line == "e" {
		break
	}

}
```

**使用 NewScanner**

```go
scanner := bufio.NewScanner(os.Stdin)
for scanner.Scan() {
	fmt.Println(scanner.Text()) // Println will add back the final '\n'
}
if err := scanner.Err(); err != nil {
	fmt.Fprintln(os.Stderr, "reading standard input:", err)
}
```

Scanner的每一次调用都会调入一个新行，并且会自动将其行末的换行符去掉；其结果可以用input.Text()得到。Scan方法在读到了新行的时候会返回true，而在没有新行被读入时，会返回false



**一个词一个词读取**

```go
scanner := bufio.NewScanner(os.Stdin)
scanner.Split(bufio.ScanWords)
for scanner.Scan() {
	fmt.Println(scanner.Text())
}
if err := scanner.Err(); err != nil {
	fmt.Fprintln(os.Stderr, "reading input:", err)
}
```

## fmt

```go
var buf string
for {
	_, err := fmt.Scanln(&buf)
	if err != nil {
		fmt.Println(err)
		break
	}
	fmt.Println(buf)
	time.Sleep(100 * time.Microsecond)
}
```

读取到空格的时候就强制返回了。

还可以定义格式：

```go
var (
	name string
	age  int
)
for {
	_, err := fmt.Scanf("%s %d", &name, &age)
	if err != nil {
		fmt.Println(err)
		break
	}
	fmt.Printf("name is %s, age is %d\n", name, age)
	time.Sleep(100 * time.Microsecond)
}
```


# 向错误流输出信息

```go
fmt.Fprint(os.Stderr,"Fatal error: %s\n",err.Error())
```

# 清屏

```go
package main

import (
	"os"
	"os/exec"
	"runtime"
)

var clear map[string]func() //create a map for storing clear funcs

func init() {
	clear = make(map[string]func()) //Initialize it
	clear["linux"] = func() {
		cmd := exec.Command("clear") //Linux example, its tested
		cmd.Stdout = os.Stdout
		cmd.Run()
	}
	clear["windows"] = func() {
		cmd := exec.Command("cls") //Windows example it is untested, but I think its working
		cmd.Stdout = os.Stdout
		cmd.Run()
	}
}

func CallClear() {
	value, ok := clear[runtime.GOOS] //runtime.GOOS -> linux, windows, darwin etc.
	if ok {                          //if we defined a clear func for that platform:
		value() //we execute it
	} else { //unsupported platform
		panic("Your platform is unsupported! I can't clear terminal screen :(")
	}
}

func main() {
	CallClear()
}

```


# 优雅退出程序（监听ctrl+c事件）

```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	signalChan := make(chan os.Signal, 1)
	signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)

	// some code

	<-signalChan
	fmt.Println("exit by program.")
	os.Exit(0)
}
```
