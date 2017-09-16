<!-- TOC -->

- [打印输出](#打印输出)
    - [复用参数](#复用参数)
    - [字符串](#字符串)
    - [字符](#字符)
    - [整型](#整型)
    - [十六进制整型](#十六进制整型)
    - [浮点型](#浮点型)
    - [输出变量类型](#输出变量类型)
    - [输出值](#输出值)
    - [输出指针](#输出指针)
    - [带有引号的字符串](#带有引号的字符串)
    - [Println输出多行文本](#println输出多行文本)
    - [自定义类型的String()](#自定义类型的string)
    - [不带fmt的输出](#不带fmt的输出)

<!-- /TOC -->


# 打印输出

对每个Printf，Fprintf和Sprintf，都有另外一对相应的函数，例如Print和Println。这些函数不接受格式串，而是为每个参数生成一个缺省的格式。Println版本还会在参数之间插入一个空格，并添加一个换行，而Print版本只有当两边的操作数都不是字符串的时候才增加一个空格。

```go
fmt.Println("hello", 23)
fmt.Print("hello", "chen", "\n")
fmt.Print("hello", 23, "\n")
fmt.Print(12, 23, "\n")
```

打印：

```
hello 23
hellochen
hello23
12 23
```

格式化打印函数fmt.Fprint等，接受的第一个参数为任何一个实现了io.Writer接口的对象；变量os.Stdout和os.Stderr是常见的实例

```go
fmt.Fprint(os.Stdout, "hello")
fmt.Fprint(os.Stderr, "chen")
```

## 复用参数

```go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // 438 666 0666
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x) // 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

%之后的[1]副词告诉Printf函数再次使用第一个操作数

%后的#副词告诉Printf在用%o、%x或%X输出时生成0、0x或0X前缀 


## 字符串

字符串前加空格

```go
fmt.Printf("|%10s|\n", "hello")
```

输出：

```
|     hello|
```

字符串后加空格

```go
fmt.Printf("|%-10s|\n", "hello")
```

数字前有个 '-'，输出：

```
|hello     |
```


## 字符

```go
var str string = "chen"
ch := str[0]
fmt.Printf("str = %s, ch = %c \n", str, ch)
```

## 整型

```go
var i int32 = 123
fmt.Printf("integer = %d \n", i)
```

在数字前用0填充

```go
fmt.Printf("%011d\n", 1234) // 00000001234
```

## 十六进制整型

%x用于字符串，字节数组和字节切片，以及整数，生成一个长的十六进制字符串，并且如果在格式中有一个空格（% x），其将会在字节中插入空格。

用于整型：
```go
var v uint64 = 1<<64 - 1
fmt.Printf("%d %x %X\n", v, v, v)
```

18446744073709551615 ffffffffffffffff FFFFFFFFFFFFFFFF

用于字符串：
```go
var v string = "hello"
fmt.Printf("%x %X\n", v, v) // 68656c6c6f 68656C6C6F
```

用于字符数组：

```go
v := []string{"ab", "cd"}
fmt.Printf("%x %X\n", v, v) // [6162 6364] [6162 6364]
```

用于字符切片：
```go
v := []string{"ab", "cd", "辰"}
slice := v[:]
fmt.Printf("%X\n", slice) // [6162 6364 E8BEB0]
```

用于整数：
```go
fmt.Printf("%X\n", 97) // 61
```

加空格：
```go
fmt.Printf("% X\n", "hello") // 68 65 6C 6C 6F
fmt.Printf("% X\n", []string{"ab", "cd"}) // [61 62 63 64]
```

## 浮点型

```go
var v1 float32 = 4.3
var v2 float64 = 5.6
fmt.Printf("%f %f\n", v1, v2) // 4.300000 5.600000
fmt.Printf("%g %g\n", v1, v2) // 4.3 5.6
```

控制输出的精度（四舍五入）：
```go
fmt.Printf("%.2f", 3.1482) // 3.15
```

## 输出变量类型

```go
i := 3
fmt.Printf("%T", i) // int

const pi = 3.14
fmt.Printf("%t", pi) // %!t(float64=3.14)
```

## 输出值

产生Print和Println的结果。
```go
i := []int{1, 2}
fmt.Printf("%v", i) // [1 2]
j := 3.4
fmt.Printf("%v", j) // 3.4

i := []int{1, 2}
fmt.Printf("%#v", i) // []int{1,2}
j := 3.4
fmt.Printf("%#v", j) // 3.4
```

当打印一个结构体时，带修饰的格式%+v会将结构体的域使用它们的名字进行注解，对于任意的值，格式%#v会按照完整的Go语法打印出该值。 
```go
type T struct {
  a int
  b float64
  c string
}
t := &T{7, 6.4, "abc\tdef"}
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
```

输出：
```go
&{7 6.4 abc	def}
&{a:7 b:6.4 c:abc	def}
&main.T{a:7, b:6.4, c:"abc\tdef"}
```

前面的&表示是指针。

## 输出指针

```go
i := 3
fmt.Printf("%d的地址是%p", i, &i) // 3的地址是0x10328000
```

## 带有引号的字符串

通过%q来实现带引号的字符串格式，用于类型为string或[]byte的值。格式%#q将尽可能的使用反引号（嵌套的引号使用不同的单双引号）。（格式%q还用于整数和符文，产生一个带单引号的符文常量。）

```go
str := []string{"abc\"hello,'chen'.\"",
  "abc'hello,\"chen\".'",
  "abc\"hello,\"chen\".\""}
for _, s := range str {
  fmt.Printf("%q %#q\n", s, s)
}
```

输出：
```
"abc\"hello,'chen'.\"" `abc"hello,'chen'."`
"abc'hello,\"chen\".'" `abc'hello,"chen".'`
"abc\"hello,\"chen\".\"" `abc"hello,"chen"."`
```

用于整型时将转换成字符：
```go
c := 97
fmt.Printf("%q %#q\n", c, c) // 'a' 'a'
```

字符串中有\t:
```go
v := "hello\tchen"
fmt.Printf("%q %#q\n", v, v) // "hello\tchen" `hello	chen`
```

## Println输出多行文本

```go
fmt.Println(`
  Enter following commands to control the player:
  lib list -- View the existing music lib
  lib add <name><artist><source><type> -- Add a music to the music lib
  lib remove <name> -- Remove the specified music from the lib
  play <name> -- Play the specified music
`)
```

输出：
```

		Enter following commands to control the player:
		lib list -- View the existing music lib
		lib add <name><artist><source><type> -- Add a music to the music lib
		lib remove <name> -- Remove the specified music from the lib
		play <name> -- Play the specified music
	
```


  "`" 这个是1前面的那个键。

  \`This is a raw string \n\` 中的 \`\n\` 会被原样输出

## 自定义类型的String()

```go
type T struct {
  a int
  b float64
  c string
}

func (t *T) String() string {
	return fmt.Sprintf("%d-%g-%q", t.a, t.b, t.c)
}

t := &T{23, 6.4, "hello"}
fmt.Printf("%v\n", t)
```

输出：
```
23-6.4-"hello"
```

不要将调用Sprintf的String方法构造成无穷递归。如果Sprintf调用尝试将接收者直接作为字符串进行打印，就会导致再次调用该方法，发生这样的情况。这是一个很常见的错误，正如这个例子所示。
```go
type MyString string

func (m MyString) String() string {
  return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

这也容易修改：将参数转换为没有方法函数的，基本的字符串类型。
```go
type MyString string
func (m MyString) String() string {
  return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

只有Sprintf想要一个字符串的时候，才去调用m的String方法，如果是”%f”这样的，就不会去调用该方法。

## 不带fmt的输出

单纯地打印一个字符串或变量甚至可以使用预定义的方法来实现，如：print、println：print("ABC")、println("ABC")、println(i)（带一个变量 i）。

这些函数只可以用于调试阶段，在部署程序的时候务必将它们替换成 fmt 中的相关函数。
