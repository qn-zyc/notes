<!-- TOC -->

- [变量](#变量)
    - [变量声明](#变量声明)
    - [变量初始化](#变量初始化)
    - [变量赋值](#变量赋值)
    - [匿名变量](#匿名变量)
    - [new()和make()](#new和make)
- [常量](#常量)
    - [字面常量](#字面常量)
    - [常量定义](#常量定义)
    - [预定义常量](#预定义常量)
    - [枚举](#枚举)

<!-- /TOC -->

# 变量
## 变量声明
```go
var v1 int
var v2 string
var v3 [10]int // 数组
var v4 []int   // 数组切片
var v5 struct {
  f int
}
var v6 *int
var v7 map[string]int
var v8 func(a int) int // 函数
```

```go
var i int8 = 0 等同于 i := (int8)(0)
var i *int8 = nil 等同于 j := (*int8)(nil)
```

```go
var min, max uint = 1, 5
var i, j, k int // 都是int，零值是0
var a, b, c = true, 2.3, "foo" // 自动推导类型
var f, err = os.Open(name) // 通过调用函数赋值
```

## 变量初始化
```go
var v1 int = 10
var v2 = 10     // 编译器自动推导v2的类型
v3 := 10        // 编译器自动推导v2的类型
```

出现在 `:=` 左侧的变量不应该是已经被声明过的，否则会导致编译错误，比如下面这个
写法：

```go
var i int 
i := 2 
```
会导致类似如下的编译错误：
```
no new variables on left side of :=
```

---------------------
`:=` 不能在函数外使用，函数外的语法块必须以关键字开始
```go
val := 4 // 错误
var val int32 = 4 // 正确
```

## 变量赋值
多重赋值功能，比如下面这个交换i和j变量的语句：
```go
i, j = j, i
```
重新声明和重新赋值
```go
a, err := f1.Close()
b, err := f2.Close()
```
err在两条语句中都出现了，第一次的err是声明，第二次的err只是重新赋值了，第二个语句中必须有至少一个变量是需要声明的，才能使用:=

------------------

**重新赋值与定义新同名变量的区别**

```go
s := "abs"
fmt.Println(&s)

s, y := "hello", 20 // 重新赋值，与前s在同一层次的代码块中，且有新的变量被定义
fmt.Println(&s, y)  // 通常函数多返回值 err 会被重复使用

{
	s, z := 1000, 30 // 定义新同名变量，不在同一层次代码块
	fmt.Println(&s, z)
}
```

输出：

	0xc08200c340
	0xc08200c340 20
	0xc08200c380 30

-------------------

**变量被重新赋值时不会改变地址**

```go
a := 100
b := "a"
fmt.Println(&a, &b)
a = 200
b = "b"
fmt.Println(&a, &b)
```

输出：

	0xc08200c3a0 0xc08200c3b0
	0xc08200c3a0 0xc08200c3b0

------------------
**平行赋值**
```go
a[i], a[j] = a[j], a[i]
```
覆盖的问题：
在进行多个变量赋值时，编译器会先计算赋值语句右边所有的值，之后再进行赋值。
```go
    a[i] =1 , a[j] = 2
    a[i], a[j] = a[j], a[i]    <=>   a[i] ,a[j] = 2, 1 
```

在赋值之前, 赋值语句右边的所有表达式将会先进行求值, 然后再统一更新左边变量的值.

------------------

> 来自[雨痕笔记]

多变量赋值时，先计算所有相关值，然后再从左到右依次赋值：

```go
data, i := [3]int{0, 1, 2}, 0
i, data[i] = 2, 100 // (i=0) -> (i=2), (data[0] = 100)
fmt.Println(data)   // [100 1 2]
```

i被赋值成2之前，data[i]已经被计算成data[0]了。

再看一个右边是否也计算的例子：

```go
data, i := [3]int{0, 2, 4}, 1
i, data[i] = data[i], 100
fmt.Println(i, data) // 2 [0 100 4]
```

也就是说在赋值之前，左右两边都计算过了。


## 匿名变量

假设GetName()函数的定义如下，它返回3个值，分别为firstName、lastName和nickName：
```go
func GetName() (firstName, lastName, nickName string) { 
	return"May", "Chan", "Chibi Maruko" 
} 
```
若只想获得nickName，则函数调用语句可以用如下方式编写：
```go
_, _, nickName := GetName()
```

## new()和make()

`new`：并不初始化内存，只是将其置零，并返回地址。如new(T)，其返回一个指向新分配的 类型为T值为零的 指针。

```go
t1 := new(T) // type *T
var t2 T     // type T
```
之后就可以直接使用t1,t2了。

但有时候需要一个初始化构造器：

```go
t := new(T)
t.f1 = v1
t.f2 = v2
```
也可以使用复合文字(composite literal):
```go
t := T{v1, v2} // type T
t := &T{v1, v2} // type *T
```
复合文字的域按顺序排列，并且必须都存在。

通过field:value显式地为元素添加标号，则初始化可以按任何顺序出现，没有出现的则对应为零值

```go
t := &T{f2:v2, f1:v1}
```
new(T)和&T{}是等价的，字段的值都是零值。

`make`：内建函数make(T, args)与new(T)的用途不一样。它*只用来创建slice，map和channel*，并且返回一个初始化的(而不是置零)，*类型为T*的值（而不是*T）。之所以有所不同，是因为这三个类型的背后是象征着，对使用前必须初始化的数据结构的引用。例如，slice是一个三项描述符，包含一个指向数据（在数组中）的指针，长度，以及容量，在这些项被初始化之前，slice都是nil的。对于slice，map和channel，make初始化内部数据结构，并准备好可用的值.
如make([]int, 10, 100)代表分配一个有100个int的数组，然后创建一个长度为10，容量为100的slice结构，并指向数组前10个元素上。而new([]int)返回一个指向新分配的，被置零的slice结构体的指针，即指向nilslice值的指针。

这些例子阐释了new和make之间的差别。

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful(很少使用)
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:（不必要的组合）
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:（符合语言习惯的）
v := make([]int, 100)
```
***记住make只用于map，slice和channel，并且不返回指针。要获得一个显式的指针，使用new进行分配，或者显式地使用一个变量的地址。***

------------------

# 常量

## 字面常量

```go
-12 
3.14159265358979323846 // 浮点类型的常量
3.2+12i // 复数类型的常量
true // 布尔类型的常量
"foo" // 字符串常量
```
Go语言的字面常量是无类型的，只要这个常量在相应类型的值域范围内，就可以作为该类型的常量，比如12，它可以赋值给int、uint、int32、int64、float32、float64、complex64、complex128等类型的变量。

## 常量定义
存储在常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型.
通过const关键字，你可以给字面常量指定一个友好的名字：
```go
const Pi float64= 3.14159265358979323846 
const zero = 0.0 // 无类型浮点常量
const (
	size int64= 1024 
	eof = -1 // 无类型整型常量
) 
const u, v float32 = 0, 3 // u = 0.0, v = 3.0，常量的多重赋值
const a, b, c = 3, 4, "foo" 
// a = 3, b = 4, c = "foo", 无类型整型和字符串常量
```

Go的常量定义可以限定常量类型，但不是必需的。如果定义常量时没有指定类型，那么它与字面常量一样，是无类型常量。
一个没有指定类型的常量被使用时，会根据其使用环境而推断出它所需要具备的类型。换句话说，未定义类型的常量会在必要时刻根据上下文来获得相关类型。
```go
var n int
f(n + 5) // 无类型的数字型常量 “5” 它的类型在这里变成了 int
```

----------

常量定义的右值也可以是一个在编译期运算的常量表达式，比如

```go
const mask = 1 << 3 
```
由于常量的赋值是一个编译期行为，所以右值不能出现任何需要运行期才能得出结果的表达式，比如试图以如下方式定义常量就会导致编译错误：
```go
const Home = os.GetEnv("HOME") 
```
原因很简单，os.GetEnv()只有在运行期才能知道返回结果，在编译期并不能确定，所以无法作为常量定义的右值。

```go
const (
	a = "abc"
	b = len(a)
	c = unsafe.Sizeof(b)
)
println(b, c) // 3 8
```

-------------------------

```go
const x = "xxx" // 未使用的局部变量不会引发编译错误
```
--------------------------

在常量组中，如不提供类型和初始化值，那么视作与上一常量相同：

```go
const (
	s = "abc"
	x
)
println(x) // abc
```

--------------------------

枚举的常量都为同一类型时, 可以使用简单序列格式

```go
const (
    a = iota    // a int32 = 0
    b            // b int32 = 1
    c            // c int32 = 2
)
```

枚举序列中的未指定类型的常量会跟随序列前面最后一次出现类型定义的类型

```go
const (
    a byte = iota    // a uint8 = 0
    b                // b uint8 = 1
    c                // c uint8 = 2
    d rune = iota    // d int32 = 3
    e                // e int32 = 4
    f                // f int32 = 5
)
```

--------

跳过一个iota

```go
	const (
		a byte = iota
		b
		_ // 跳过
		c
	)
	fmt.Println(a, b, c) // 0 1 3
```

-----------

在同一常量组中，可以提供多个iota，他们各自增长：

```go
const (
	a, b = iota, iota << 10 // 0, 0<<10
	c, d                    // 1, 1<<10
)
println(a, b, c, d) // 0 0 1 1024
```

----------------------

如果iota自增被打断，须显示恢复

```go
const (
	a = iota // 0
	b        // 1
	c = "c"  // c
	d        // c，与上一行相同
	e = iota // 4, 显示恢复，注意计数包含了 c, d 两行
	f        // 5
)
```

注意e的值

--------------------------

自定义函数不能为常量赋值

```go
const (
	a = iota
	//b = getNumber() 错误
	c = len([3]int{})
)

func getNumber() int {
	return 1
}
```
因为在编译期间自定义函数均属于未知，因此无法用于常量的赋值，但内置函数可以使用，如：len() 。

-------------
数字型的常量是没有大小和符号的，并且可以使用任何精度而不会导致溢出
```go
const Ln2= 0.693147180559945309417232121458\
            176568075500134360255254120680009
const Log2E= 1/Ln2 // this is a precise reciprocal
const Billion = 1e9 // float constant
const hardEight = (1 << 100) >> 97
```
反斜杠 \ 可以在常量表达式中作为多行的连接符使用.
不过需要注意的是，当常量赋值给一个精度过小的数字型变量时，可能会因为无法正确表达常量所代表的数值而导致溢出，这会在编译期间就引发错误.

-----------------------

可以通过自定义类型来实现枚举类型限制：

```go
type Color int
```

```go
const (
	Black Color = iota
	Red
	Blue
)

func test(c Color) {}

func main() {
	c := Black
	test(c)

	//	x := 1
	//	test(x) // Error: cannot use x (type int) as type Color in argument to test

	test(1) // 常量会被编译器自动转换
}
```


## 预定义常量

Go语言预定义了这些常量：true、false和iota
iota比较特殊，可以被认为是一个可被编译器修改的常量，在每一个const关键字出现时被重置为0，然后在下一个const出现之前，每出现一次iota，其所代表的数字会自动增1。
从以下的例子可以基本理解iota的用法：
```go
const(    // iota被重设为0
	c0 = iota  // c0 == 0 
	c1 = iota  // c1 == 1 
	c2 = iota  // c2 == 2 
) 
const( 
	a = 1 << iota   // a == 1 (iota在每个const开头被重设为0) 
	b = 1 << iota   // b == 2 
	c = 1 << iota   // c == 4 
) 
const( 
	u = iota * 42  // u == 0 
	v float64 = iota * 42  // v == 42.0 
	w = iota * 42  // w == 84 
) 
constx = iota  // x == 0 (因为iota又被重设为0了) 
consty = iota  // y == 0 (同上) 
```
```go
	const (
		e, f, g = iota, iota, iota //e=0,f=0,g=0 iota在同一行值相同
	)
```

如果两个const的赋值语句的表达式是一样的，那么可以省略后一个赋值表达式。因此，上面的前两个const语句可简写为：

```go
const(      // iota被重设为0
	c0 = iota  // c0 == 0 
	c1       // c1 == 1 
	c2      // c2 == 2 
) 
const( 
	a = 1 << iota    // a == 1 (iota在每个const开头被重设为0) 
	b       // b == 2 
	c       // c == 4 
) 
```

## 枚举
Go语言并不支持众多其他语言明确支持的enum关键字。
下面是一个常规的枚举表示法，其中定义了一系列整型常量：

```go
const( 
	Sunday = iota
	Monday 
	Tuesday 
	Wednesday 
	Thursday 
	Friday 
	Saturday 
	numberOfDays  // 这个常量没有导出
) 
```
同Go语言的其他符号（symbol）一样，以大写字母开头的常量在包外可见。
以上例子中numberOfDays为包内私有，其他符号则可被其他包访问

