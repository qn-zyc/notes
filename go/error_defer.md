<!-- TOC -->

- [error](#error)
    - [Go中error类型的nil值和nil](#go中error类型的nil值和nil)
- [返回错误信息](#返回错误信息)
- [自定义错误类型](#自定义错误类型)
- [defer 延迟函数](#defer-延迟函数)
- [panic和recover](#panic和recover)
    - [看一下 json 包中的一段代码](#看一下-json-包中的一段代码)
- [错误处理的冗余](#错误处理的冗余)
- [提供更友好的错误提示](#提供更友好的错误提示)

<!-- /TOC -->


## error

error类型的声明可在`builtin`包中查看：

```go
type error interface {
    Error() string
}
```

如果直接使用这个，我们需要声明一个结构体然后实现Error()方法，常用的是errors包中的New()：

```go
// New returns an error that formats as the given text.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```


### Go中error类型的nil值和nil

**问题描述**

```go
func retErr() error {
	var r *MyError = nil
	if err() {
		r = &MyError{}
	}
	return r
}
func main() {
	err := retErr()
	if err != nil {
		fmt.Println("error:", err)
	} else {
		fmt.Println("no error.")
	}
}
```

这里retErr()返回的err并不是nil.

> 在底层，interface作为两个成员实现：一个类型和一个值。该值被称为接口的动态值， 它是一个任意的具体值，而该接口的类型则为该值的类型。

> 只有在内部值和类型都未设置时(nil, nil)，一个接口的值才为 nil。特别是，一个 nil 接口将总是拥有一个 nil 类型。若我们在一个接口值中存储一个 int 类型的指针，则内部类型将为 int，无论该指针的值是什么：(*int, nil)。 因此，这样的接口值会是非 nil 的，即使在该指针的内部为 nil。

所以，上面的err是一个有效值（非nil），值为 nil。

**处理方式一**

retErr()在返回值时，如果要返回nil，就直接return nil。

**处理方式二**

main()在判断时这样：

```go
if err.(*MyError) != nil
```



## 返回错误信息

```go
import (
	"errors"
	"fmt"
)

func main() {
	ret, err := error_test(false)
	if err != nil {
		fmt.Println("err is ", err, "ret is ", ret)
	} else {
		fmt.Println("ret is ", ret, "err is ", err)
	}
}

func error_test(flag bool) (ret string, err error) {
	if flag {
		err = errors.New("error test")
		return
	} else {
		return "success", nil
	}
}
```

若传递true,结果为`err is  error test ret is  `

若传递false,结果为`ret is  success err is  <nil>`

## 自定义错误类型

Go语言引入了一个关于错误处理的标准模式，即error接口，该接口的定义如下：

```go
type error interface{ 
	Error() string
} 
```

下面红色部分代码参考src\pkg\os\error.go

```go
import (
	"errors"
	"fmt"
)

func main() {
	err := my_error()
	if err != nil {
		fmt.Println("err is ", err)
	} else {
		fmt.Println("err is nil")
	}
}
func my_error() error {
	_, err := error_test(true)
	if err != nil {
		return &PathError{"test", "path", err}
	} else {
		return nil
	}
}

func error_test(flag bool) (ret string, err error) {
	if flag {
		err = errors.New("error test")
		return
	} else {
		return "success", nil
	}
}

type PathError struct {
	Op   string
	Path string
	Err  error
}

func (e *PathError) Error() string {
	return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

## defer 延迟函数

它的作用是：延迟执行，在声明时不会立即执行，而是在函数return后时按照后进先出的原则依次执行每一个defer。这样带来的好处是，能确保我们定义的函数能百分之百能够被执行到，这样就能做很多我们想做的事，如释放资源，清理数据，记录日志等

```go
func CopyFile(dst, src string) (w int64, err error) {
	srcFile, err := os.Open(src)
	if err != nil {
		return
	}
	defer srcFile.Close()
	dstFile, err := os.Create(dstName)
	if err != nil {
		return
	}
	defer dstFile.Close()
	return io.Copy(dstFile, srcFile)
}
```

即使其中的Copy()函数抛出异常，Go仍然会保证dstFile和srcFile会被正常关闭。

如果觉得一句话干不完清理的工作，也可以使用在defer后加一个匿名函数的做法：

	defer func() {
		// 做你复杂的清理工作
	}()

defer语句的调用是遵照先进后出的原则，即最后一个defer语句将最先被执行

------------------

下面说明一下defer执行的顺序

```go
func deferFunc() int {
	index := 0

	fc := func() {

		fmt.Println(index, "匿名函数1")
		index++

		defer func() { 
			fmt.Println(index, "匿名函数1-1")
			index++
		}()
	}

	defer func() { 
		fmt.Println(index, "匿名函数2")
		index++
	}()

	defer fc()

	return func() int {
		fmt.Println(index, "匿名函数3")
		index++
		return index
	}()
}

func main() {
	ret := deferFunc()
	fmt.Println("main() :", ret)
}
```

执行结果：

```
0 匿名函数3
1 匿名函数1
2 匿名函数1-1
3 匿名函数2
main() : 1
```

1. defer是在执行完return后执行
2. return的结果并没有随着index递增
3. 按照先进后出的顺序执行


------------------

返回语句之后的defer不会执行

```go
func main() {
	i := true

	defer func() {
		fmt.Println("f1")
	}()

	defer func() {
		fmt.Println("f2")
	}()

	if i {
		return
	}

	defer func() {
		fmt.Println("f3")
	}()
}
```

输出： f2 f1,没有f3

---------------------

被延期执行的函数，它的参数（包括接收者，如果函数是一个方法）是在defer执行的时候被求值的，而不是在调用执行的时候。这样除了不用担心变量随着函数的执行值会改变，这还意味着单个被延期执行的调用点可以延期多个函数执行。这里有一个简单的例子

	for i := 0; i < 3; i++ {
		defer fmt.Print(i, " ")
	}

输出：2 1 0

-------------------

验证defer的函数的参数是在defer定义的地方被求值的

```go
func enter(s string) string {
	fmt.Println("entering ", s)
	return s
}
func leave(s string) {
	fmt.Println("leaving ", s)
}
func a() {
	defer leave(enter("a"))
	fmt.Println("in a...")
	b()
}
func b() {
	defer leave(enter("b"))
	fmt.Println("in b...")
}
```

main()中调用a()

输出：

```
entering  a
in a...
entering  b
in b...
leaving  b
leaving  a
```

被延迟的函数是leave，它的参数是enter的返回值，所以在定义defer的地方先求了enter的返回值，上面defer的结果等同于下面的语句：

```go
enter(“a”)
defer leave(“a”)
```

--------------------

defer中改变返回值

```go
	f := func() (ret int) {
		defer func() {
			ret++
		}()
		return 1
	}
	v := f()
	fmt.Println(v)
```

输出2.


## panic和recover

```go
func panic(interface{}) 
func recover() interface{}
```

当在一个函数执行过程中调用panic()函数时，正常的函数执行流程将立即终止，但函数中之前使用defer关键字延迟执行的语句将正常展开执行，之后该函数将返回到调用函数，并导致逐层向上执行panic流程，直至所属的goroutine中所有正在执行的函数被终止。错误信息将被报告，包括在调用panic()函数时传入的参数，这个过程称为错误处理流程

panic类似于抛出异常，recover类似于捕获异常

假如执行：

	func() {
		panic("panic text.")
	}()

程序将抛出错误然后退出，但如果这样：

	defer func() {
		if r := recover(); r != nil {
			fmt.Println("recover text: ", r)
		}
	}()
	
	func() {
		panic("panic text.")
	}()
	
	fmt.Println("panic之后的语句")

程序执行的结果为：

```
recover text:  panic text.
```

panic()之后，程序就退出函数了，之后的语句不会执行了，除了defer

如果没有panic,上面定义的defer也会在函数退出时执行。

### 看一下 json 包中的一段代码

```go
func (d *decodeState) unmarshal(v interface{}) (err error) {
	defer func() {
		if r := recover(); r != nil {
			if _, ok := r.(runtime.Error); ok {
				panic(r) // 这里
			}
			err = r.(error)
		}
	}()

	rv := reflect.ValueOf(v)
	if rv.Kind() != reflect.Ptr || rv.IsNil() {
		return &InvalidUnmarshalError{reflect.TypeOf(v)}
	}

	d.scan.reset()
	// We decode rv not rv.Elem because the Unmarshaler interface
	// test must be applied at the top level of the value.
	d.value(rv)
	return d.savedError
}
```

这个实现中，我们可以获知几个用法：

- 在恢复时，通过函数的命名返回值可以返回错误信息
- 通过类型断言可以判断是运行时错误还是调用了 panic 函数
- 在恢复时，可以继续调用 panic


## 错误处理的冗余

```go
func init() {
    http.HandleFunc("/view", viewRecord)
}

func viewRecord(w http.ResponseWriter, r *http.Request) {
    c := appengine.NewContext(r)
    key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
    record := new(Record)
    if err := datastore.Get(c, key, record); err != nil {
        http.Error(w, err.Error(), 500)
        return
    }
    if err := viewTemplate.Execute(w, record); err != nil {
        http.Error(w, err.Error(), 500)
    }
}
```

每次发生错误都需要调用`http.Error(w, err.Error(), 500)`，还可能有更复杂多处理逻辑。

通过复用检测函数来减少类似的代码

```go
type appHandler func(http.ResponseWriter, *http.Request) error

func (fn appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if err := fn(w, r); err != nil {
        http.Error(w, err.Error(), 500)
    }
}
```

这里的appHandler和上面的viewRecord实现了相同的接口，用appHandler来包装viewRecord。

注册时这样：

```go
func init() {
    http.Handle("/view", appHandler(viewRecord))
}
```

然后viewRecord就可以这样写：

```go
func viewRecord(w http.ResponseWriter, r *http.Request) error {
    c := appengine.NewContext(r)
    key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
    record := new(Record)
    if err := datastore.Get(c, key, record); err != nil {
        return err
    }
    return viewTemplate.Execute(w, record)
}
```

其实是将错误处理的共有代码封装到了appHandler函数中。

## 提供更友好的错误提示

自定义错误类型

```go
type appError struct {
    Error   error
    Message string
    Code    int
}
```

这样我们的自定义路由器可以改成如下方式：

```go
type appHandler func(http.ResponseWriter, *http.Request) *appError

func (fn appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if e := fn(w, r); e != nil { // e is *appError, not os.Error.
        c := appengine.NewContext(r)
        c.Errorf("%v", e.Error)
        http.Error(w, e.Message, e.Code)
    }
}
```

这样修改完自定义错误之后，我们的逻辑处理可以改成如下方式：

```go
func viewRecord(w http.ResponseWriter, r *http.Request) *appError {
    c := appengine.NewContext(r)
    key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
    record := new(Record)
    if err := datastore.Get(c, key, record); err != nil {
        return &appError{err, "Record not found", 404}
    }
    if err := viewTemplate.Execute(w, record); err != nil {
        return &appError{err, "Can't display record", 500}
    }
    return nil
}
```



