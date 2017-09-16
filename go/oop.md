<!-- TOC -->

- [类型系统](#类型系统)
    - [为类型添加新方法](#为类型添加新方法)
    - [值语义和引用语义](#值语义和引用语义)
        - [数组传递也是值传递](#数组传递也是值传递)
        - [Go语言中有4个类型比较特别，看起来像引用类型](#go语言中有4个类型比较特别看起来像引用类型)
        - [自定义引用类型](#自定义引用类型)
- [结构体](#结构体)
    - [定义](#定义)
    - [初始化](#初始化)
    - [匿名结构体和结构体数组](#匿名结构体和结构体数组)
    - [空结构体地址相同](#空结构体地址相同)
    - [结构体数组可以不需要写结构体名了](#结构体数组可以不需要写结构体名了)
- [匿名组合](#匿名组合)
- [隐藏结构体的方法](#隐藏结构体的方法)
- [浅复制](#浅复制)
- [可见性](#可见性)
- [接口](#接口)
    - [非侵入式接口](#非侵入式接口)
    - [接口赋值(对象实例赋值给接口)](#接口赋值对象实例赋值给接口)
    - [接口赋值(接口赋值给接口)](#接口赋值接口赋值给接口)
    - [接口查询](#接口查询)
    - [类型查询](#类型查询)
    - [接口组合](#接口组合)
    - [Any类型](#any类型)
    - [验证实现类是否实现了接口](#验证实现类是否实现了接口)
- [Go 语言方法接受者类型的选择](#go-语言方法接受者类型的选择)
    - [概述](#概述)
    - [何时使用值类型](#何时使用值类型)
    - [何时使用指针类型](#何时使用指针类型)
    - [关于接受者的命名](#关于接受者的命名)
- [Go 语言中的方法，接口和嵌入类型](#go-语言中的方法接口和嵌入类型)
    - [概述](#概述-1)
    - [方法](#方法)
    - [接口](#接口-1)
    - [实现接口](#实现接口)
    - [嵌入类型](#嵌入类型)
    - [回答开头的问题](#回答开头的问题)
    - [总结](#总结)
- [接口的类型和值](#接口的类型和值)
- [方法接收者是值/引用的一个小例子](#方法接收者是值引用的一个小例子)

<!-- /TOC -->


go没有面向对象的概念，所以也没有继承概念，有的只是组合。

# 类型系统

## 为类型添加新方法

```go
func main() {
	var v Integer = 1
	fmt.Println(v.Less(2)) // true
}

type Integer int

func (a Integer) Less(b Integer) bool {
	return a < b
}
```

**Integer是一个新类型（也可以当做int的别名），它和int没有本质不同，只是它为内置的int类型增加了个新方法Less()。这里并不是真正意义上的别名，因为使用这种方法定义之后的类型可以拥有更多的特性，且在类型转换时必须显式转换.**

-----------------

int 和 Integer 可以相互转换： `Integer(23); int(Integer(23));` ， 而且**这种转换并不会创建新的值**。

-----------------

**只有需要修改对象的时候，才必须用指针**

```go
func main() {
	var v Integer = 1
	v.Add(3)
	fmt.Println(v) // 4
}

type Integer int

func (a *Integer) Add(b Integer) {
	*a += b // 等同 *a = *a + b
}
```

如果没有指针：

```go
func main() {
	var v Integer = 1
	v.Add(3)
	fmt.Println(v) // 1
}

type Integer int

func (a Integer) Add(b Integer) {
	a += b
}
```

**Go语言和C语言一样，类型都是基于值传递的。要想修改变量的值，只能传递指针。**

**使用指针的Append**

```go
type ByteSlice []byte

func (s *ByteSlice) Append(data byte) {
	slice := *s
	slice = append(slice, data)
	*s = slice
}

var s ByteSlice = []byte{1, 2, 3}
(&s).Append(4)
fmt.Println(s) // [1 2 3 4]
```

**nil指针也可以调用方法**

bytes.Buffer中声明了一个方法：

```go
func (b *Buffer) String() string {
	if b == nil {
		// Special case, useful in debugging.
		return "<nil>"
	}
	return string(b.buf[b.off:])
}
```

当这样调用时：

```go
var buf *bytes.Buffer = nil
fmt.Println(buf.String())
```

并不会报错，而是输出 `<nil>`，也就是说buf为空指针时也是可以调用方法的。而如果buf的类型是bytes.Buffer而不是指针时就会直接报错。

**如果你有多个类型需要定义，可以使用因式分解关键字的方式，例如：**

```go
type (
    IZ int
    FZ float
    STR string
)
```

每个值都必须在经过编译后属于某个类型（编译器必须能够推断出所有值的类型），因为 Go 语言是一种静态类型语言.

**这种方式并不拥有原类型的方法，但拥有原类型的字段**

```go
func main() {
	s := &Sun{}
//	s.BaseFun() // 错误，没有该方法
	fmt.Println(s.f) // 正确，有f字段
}

type Base struct {
	f string
}

func (this *Base) BaseFun() {
	fmt.Println("BaseFun")
}

type Sun Base
```

## 值语义和引用语义

Go语言中的大多数类型都基于值语义，包括：

基本类型，如byte、int、bool、float32、float64和string等； 

复合类型，如数组（array）、结构体（struct）和指针（pointer）等

### 数组传递也是值传递

	a := [3]int{1, 2, 3}
	b := a
	b[0]++
	fmt.Println(a) // [1 2 3]
	fmt.Println(b) // [2 2 3]

如果b是a的指针类型的话：

	a := [3]int{1, 2, 3}
	b := &a // 或者var b = &a
	b[0]++
	fmt.Println(a) // [2 2 3]
	fmt.Println(b) // &[2 2 3]
	fmt.Println(*b) // [2 2 3]

此时b的类型不是`[3]int`,而是`*[3]int`

### Go语言中有4个类型比较特别，看起来像引用类型

数组切片：指向数组（array）的一个区间。

map：极其常见的数据结构，提供键值查询能力。

channel：执行体（goroutine）间的通信设施。

接口（interface）：对一组满足某个契约的类型的抽象

这些类型的定义中有指针类型的字段：

```go
type slice struct { 
	first *T 
	len int 
	cap int 
}
```

### 自定义引用类型

```go
type MyType struct {
	val *int
}
```

# 结构体

Go语言的结构体（struct）和其他语言的类（class）有同等的地位，但Go语言放弃了包括继承在内的大量面向对象特性，只保留了组合（composition）这个最基础的特性

## 定义

```go
type Rect struct {
	x, y, width, height float64
}

func (r *Rect) Area() float64 {
	return r.width * r.height
}
```

## 初始化

```go
rect1 := new(Rect)
rect2 := &Rect{}
//rect3 := &Rect{0, 0} // 错误
rect4 := &Rect{0, 0, 5, 5}
rect5 := &Rect{width: 20, height: 20}
```

&rect{n}的类型是指针类型，如果不想获得指针，可以去掉&:

```go
rect := Rect{}
var rect Rect = Rect{}
```

除了第一个外，其余都是花括号`{}`

要么都不赋值（有默认值），要么都赋值，只想给一部分赋值的话需要标明字段名

另一种方式

```go
type Person struct {
	name string
	age  int
}

var p Person // 这不只是声明了还初始化了？？？
fmt.Println(p) // { 0}
p.name, p.age = "chen", 23
fmt.Println(p) // {chen 23}
```

--------------------

重复的键将导致报错：

```go
p := Persion{
	Name: "a",
	Name: "a",
}
```

报错信息：

	duplicate field name in struct literal: Name
​	


## 匿名结构体和结构体数组

```go
	var v = struct {
		a, b string
	}{"aa", "bb"} // 声明的同时初始化

	fmt.Println(v) // {aa bb}

	var vv = []struct {
		a, b string
	}{
		{"a1", "b1"},
		{"a2", "b2"},
	}
	fmt.Println(vv) // [{a1 b1} {a2 b2}]
```

```go
type Point struct {
	x, y int
}

func main() {
	points := []Point{
		{1, 2},
		{3, 4},
	}
	fmt.Println(points)
}
```


先声明，延后初始化

```go
	var inner struct {
		E interface{}
	}
	inner.E = 123
```


## 空结构体地址相同

```go
type empty struct {
}
e1 := &empty{}
e2 := &empty{}
fmt.Printf("%p, %p\n", e1, e2)
```

输出 

```
0x5b5f60, 0x5b5f60
```

如果empty里有字段，那地址就不同了。可能是为了优化，如果结构体里没有字段只有方法，那么两个结构体的执行结果肯定相同，所以共用一个地址了。


## 结构体数组可以不需要写结构体名了

```go
type Field struct {
	name string
}

fields := []Field{
	{"a"}, {"b"}, {"c"},
}

或者：

fields := []*Field{
	{"a"}, {"b"}, {"c"},
}
```

------------------------


# 匿名组合

确切地说，Go语言也提供了继承，但是采用了组合的文法，所以我们将其称为匿名组合

```go
type Base struct {
	Name string
}

func (b *Base) Foo() {
	fmt.Printf("Base.Foo():%s\n", b.Name)
}
func (b *Base) Bar() {
	fmt.Printf("Base.Bar():%s\n", b.Name)
}

type Sub struct {
	Base
	Name string
}

func (s *Sub) Foo() {
	s.Base.Foo() // 调用Base的方法
	fmt.Printf("Sub.Foo():%s\n", s.Name)
}

func main() {
	base := &Base{"i am base."}
	base.Foo()
	base.Bar()

	sub := &Sub{*base, "i am sub."} // 因为base是指针，这里要加*
	sub.Foo()
	sub.Base.Bar()
	sub.Bar() // 其实调用的是Base的方法

	fmt.Println(sub.Name)

}
```

输出：

```
Base.Foo():i am base.
Base.Bar():i am base.
Base.Foo():i am base.
Sub.Foo():i am sub.
Base.Bar():i am base.
Base.Bar():i am base.
i am sub.
```

这是组合，不是继承，只是可以看成是继承，下面把Sub赋值给Base是错误的：

```go
var v *Base = sub
```

提示：

```
cannot use sub (type *Sub) as type *Base in assignment
```

-------------

Sub会继承Base的属性和方法，可以当成自己的使用，也可以通过类型名来访问，就像java中的super

```go
type Person struct {
	string
	age int
}
p := Person{"person", 11}
fmt.Println(p.age, p.string) // 11 person,直接通过类型名访问
```

Sub没有Bar()方法，但还是可以调用Base的Bar()，调用形式看起来像是继承

----------------

当我们内嵌一个类型时，该类型的所有方法会变成外部类型的方法，但是当这些方法被调用时，其接收的参数仍然是内部类型，而非外部类型。

-----------------

Sub中Base是值类型，这样当在Sub中修改了Base.Name, base是不会有反应的

```go
sub.Base.Name = "modify"
fmt.Println(base.Name) // 还是i am base.
```

---------------

还可以继承Base指针

```go
type Sub struct {
	*Base
	Name string
}

/* 重写方法 */
func (s *Sub) Foo() {
	s.Base.Foo() // 调用Base的方法
	fmt.Printf("Sub.Foo():%s\n", s.Name)
}

func main() {
	base := &Base{"i am base."}
	base.Foo()
	base.Bar()

	sub := &Sub{base, "i am sub."}
	sub.Foo()
	sub.Base.Name = "modify"
	fmt.Println(base.Name) // modify
}
```

修改了sub中的Base，base也被同步了

-----------------

在Go语言官方网站提供的Effective Go中曾提到匿名组合的一个小价值，值得在这里再提一下。首先我们可以定义如下的类型，它匿名组合了一个log.Logger指针：

```go
type Job struct{ 
	Command string
	*log.Logger 
} 
```

在合适的赋值后，我们在Job类型的所有成员方法中可以很舒适地借用所有log.Logger提供的方法。比如如下的写法：

```go
func(job *Job)Start() { 
	job.Log("starting now...") 
	... // 做一些事情
	job.Log("started.") 
} 
```

对于Job的实现者来说，他甚至根本就不用意识到log.Logger类型的存在，这就是匿名组合的魅力所在。在实际工作中，只有合理利用才能最大发挥这个功能的价值。

需要注意的是，不管是非匿名的类型组合还是匿名组合，被组合的类型所包含的方法虽然都升级成了外部这个组合类型的方法，但其实它们被组合方法调用时接收者并没有改变。比如上面这个Job例子，即使组合后调用的方式变成了job.Log(...)，但Log函数的接收者仍然是log.Logger指针，因此**在Log中不可能访问到job的其他成员方法和变量**。
这其实也很容易理解，毕竟被组合的类型并不知道自己会被什么类型组合，当然就没法在实现方法时去使用那个未知的“组合者”的功能了

------------------

匿名组合类型相当于以其类型名称（去掉包名部分）作为成员变量的名字

```go
type Logger struct{ 
	Level int
} 
type Y struct{ 
	*Logger 
	Name string
	*log.Logger 
}
```

Y类型中就相当于存在两个名为Logger的成员，虽然类型不同。因此，我们预期会收到编译错误。有意思的是，这个编译错误并不是一定会发生的。假如这两个Logger在定义后再也没有被用过，那么编译器将直接忽略掉这个冲突问题，直至开发者开始使用其中的某个Logger



# 隐藏结构体的方法

来源：io包下io_test.go的Buffer。

```go
func main() {
	var a *A = new(A)
	a.f()
	var b *B = new(B)
	b.f()
}

type A struct {
}

func (this A) f() {
	fmt.Println("f() from A")
}

type I interface {
	f()
}

type B struct {
	A
	//I // 如果放开，调用b.f()会报错
}
```


# 浅复制

```go
type A struct {
	a string
	b int
	c map[string]string
}

func main() {
	a := &A{"a", 1, map[string]string{"a": "a"}}
	b := new(A)
	*b = *a // 浅复制
	fmt.Println(a, b)
	a.c["b"] = "b" // 两个都改变了
	a.b = 2        // 只有a改变了
	fmt.Println(a, b)
}
```

输出：

```
&{a 1 map[a:a]} &{a 1 map[a:a]}
&{a 2 map[b:b a:a]} &{a 1 map[a:a b:b]}
```



# 可见性
要使某个符号对其他包（package）可见（即可以访问），需要将该符号定义为以大写字母开头，小写字母开头的仅对包内可见，包括包内其他的类型。


# 接口

## 非侵入式接口

在Go语言中，一个类只需要实现了接口要求的所有函数，我们就说这个类实现了该接口

```go
type IFile interface {
	Read(buf []byte) (n int, err error)
	Write(buf []byte) (n int, err error)
	Close() error
}

type IRead interface {
	Read(buf []byte) (n int, err error)
}

type File struct {
	// ...
}

func (f *File) Read(buf []byte) (n int, err error) {
	fmt.Println("File.Read()")
	return 0, nil
}
func (f *File) Write(buf []byte) (n int, err error) {
	fmt.Println("File.Write()")
	return 0, nil
}
func (f *File) Close() error {
	fmt.Println("File.Read()")
	return nil
}

func main() {
	var file1 IFile = new(File)
	file1.Read([]byte{})
	file1.Write([]byte{})
	file1.Close()

	var file2 IRead = new(File)
	file2.Read([]byte{})
	//file2.Close() // 错误，未定义
}
```

## 接口赋值(对象实例赋值给接口)

定义实现类

```go
type Integer int

func (a Integer) Less(b Integer) bool {
	return a < b
}
func (a *Integer) Add(b Integer) {
	*a += b
}
```

再定义接口（这样接口就得事先知道实现类，不然不知道有Integer??）

```go
type LessAdder interface {
	Less(b Integer) bool
	Add(b Integer)
}
```

赋值

```go
var a Integer = 1 
var b LessAdder = &a ... (1) 
var b LessAdder = a ... (2) 错误
```

原因在于，Go语言可以根据下面的函数：

```go
func(a Integer) Less(b Integer) bool
```

自动生成一个新的Less()方法：

```go
func(a *Integer) Less(b Integer) bool{ 
	return(*a).Less(b) 
} 
```

这样，类型*Integer就既存在Less()方法，也存在Add()方法，满足LessAdder接口。而从另一方面来说，根据

```go
func(a *Integer) Add(b Integer)
```

这个函数无法自动生成以下这个成员方法：

```go
func(a Integer) Add(b Integer) { 
	(&a).Add(b) 
} 
```

因为(&a).Add()改变的只是函数参数a，对外部实际要操作的对象并无影响，这不符合用户的预期。所以，Go语言不会自动为其生成该函数。因此，类型Integer只存在Less()方法，缺少Add()方法，不满足LessAdder接口，故此上面的语句(2)不能赋值

为了进一步证明以上的推理，我们不妨再定义一个Lesser接口，如下：

```go
type Lesser interface{ 
	Less(b Integer) bool
} 
```

然后定义一个Integer类型的对象实例，将其赋值给Lesser接口：

```go
vara Integer = 1 
varb1 Lesser = &a ... (1) 
varb2 Lesser = a ... (2)
```

正如我们所料的那样，语句(1)和语句(2)均可以编译通过

## 接口赋值(接口赋值给接口)

在Go语言中，只要两个接口拥有相同的方法列表（次序不同不要紧），那么它们就是等同的，可以相互赋值
下面我们来看一个示例，这是第一个接口：

```go
package one 
type ReadWriter interface{ 
	Read(buf []byte) (n int, err error) 
	Write(buf []byte) (n int, err error) 
} 
```

第二个接口位于另一个包中：

```go
package two 
type IStream interface{ 
	Write(buf []byte) (n int, err error) 
	Read(buf []byte) (n int, err error) 
}
```

以下这些代码可编译通过：

```go
var file1 two.IStream = new(File) 
var file2 one.ReadWriter = file1 
var file3 two.IStream = file2
```

接口赋值并不要求两个接口必须等价。如果接口A的方法列表是接口B的方法列表的子集，那么接口B可以赋值给接口A

例如，假设我们有Writer接口：

```go
type Writer interface{ 
	Write(buf []byte) (n int, err error) 
} 
```

就可以将上面的one.ReadWriter和two.IStream接口的实例赋值给Writer接口：

```go
var file1 two.IStream = new(File) 
var file4 Writer = file1
```

但反过来并不成立


## 接口查询
定义两个接口和一个实现类

```go
type Intf1 interface {I
	F()
}

type Intf2 interface {
	F()
}

type Impl1 struct{}

func (a Impl1) F() {
	fmt.Println("F()")
}
```

查询Intf1指向的对象是否也实现了Intf2:

```go
var intf1 Intf1 = Impl1{}
if what, ok := intf1.(Intf2); ok {
	fmt.Println("ok")
	fmt.Println(what == intf1)
} else {
	fmt.Println("no")
}
```

输出：   

	ok   
	true   

如果intf1.(Intf2)中的intf1不是接口类型，会报错:

```go
var intf1 Impl1 = Impl1{} // 这样不行
```

判断是否是字符串

```go
var s interface{} // s的类型一定要是接口
s = "hello"
if _, ok := s.(string); ok {
	fmt.Println("is string")
} else {
	fmt.Println("not string")
}
```

也可以不写ok : `var.(type)`, 但是如果无法转换的话就直接panic了。


## 类型查询

在Go语言中，还可以更加直截了当地询问接口指向的对象实例的类型

```go
var intf1 interface{} = Impl1{}
switch intf1.(type) { // 类型查询
case int:
	fmt.Println("int")
case string:
	fmt.Println("string")
case Intf1: 
	fmt.Println("intf1...")
default: // 配合接口查询
	if v, ok := intf1.(Intf1); ok {
		fmt.Printf("Intf1,%T\n", v)
	}
	if v, ok := intf1.(Impl1); ok {
		fmt.Printf("Impl1,%T\n", v)
	}
}
```

输出：

	Intf1,intf.Impl1
	Impl1,intf.Impl1

注意：

1. ‘.’之前的必须是接口类型
2. intf1.(Intf1)将Interface{}类型的转换为了Intf1,因此可以写成
```go
  		if v, ok := intf1.(Intf1); ok {
  			fmt.Printf("Intf1,%T\n", v)
  			v.F()
  		}
```
3. 类型switch中无法使用fallthrough
```go
  	case Intf1:
  		fmt.Println("intf1...")
  		fallthrough
```
  报：cannot fallthrough in type switch

4. switch 中直接拿到具体类型：

   ```go
   	var i interface{} = 1
   	switch v := i.(type) {
   	case int:
   		fmt.Printf("%T", v) // int
   	}
   ```

   ​

## 接口组合

```go
// ReadWriter接口将基本的Read和Write方法组合起来
type ReadWriter interface{ 
	Reader 
	Writer 
}
```

这个接口组合了Reader和Writer两个接口，它完全等同于如下写法：

```go
type ReadWriter interface{ 
	Read(p []byte) (n int, err error) // Reader接口的方法
	Write(p []byte) (n int, err error) // Writer接口的方法
} 
```

## Any类型

由于Go语言中任何对象实例都满足空接口interface{}，所以interface{}看起来像是可以指向任何对象的Any类型，如下：

```go
varv1 interface{} = 1 // 将int类型赋值给interface{} 
varv2 interface{} = "abc" // 将string类型赋值给interface{} 
varv3 interface{} = &v2 // 将 *interface{}类型赋值给interface{} 
varv4 interface{} = struct{ X int}{1} // 声明加初始化，匿名类？
varv5 interface{} = &struct{ X int}{1} 
```

当函数可以接受任意的对象实例时，我们会将其声明为interface{}，最典型的例子是标准库fmt中PrintXXX系列的函数，例如：

```go
func Printf(fmt string, args ...interface{}) 
func Println(args ...interface{}) 
...
```


## 验证实现类是否实现了接口

```go
type Intf interface {
	Name() string
}

type Imp struct{}

func (i *Imp) Name() string {
	return "implement"
}

var _ Intf = &Imp{}
```

使用 `_` 将变量丢掉，这样就可以在编译时验证Imp是否实现了Intf。

`var _ Intf = &Imp{}` 可以写在正式代码中，也可以写在测试代码中。

还可以这样：`var _ Intf = (*Imp)(nil)`




# Go 语言方法接受者类型的选择

## 概述

很多人(特别是新手)在写 Go 语言代码时经常会问一个问题，那就是一个方法的接受者类型到底应该是值类型还是指针类型呢，Go 的 wiki 上对这点做了很好的解释，我来翻译一下。

## 何时使用值类型

- 如果接受者是一个 map，func 或者 chan，使用值类型(因为它们本身就是引用类型)。
- 如果接受者是一个 slice，并且方法不执行 reslice 操作，也不重新分配内存给 slice，使用值类型。
- 如果接受者是一个小的数组或者原生的值类型结构体类型(比如 time.Time 类型)，而且没有可修改的字段和指针，又或者接受者是一个简单地基本类型像是 int 和 string，使用值类型就好了。

一个值类型的接受者可以减少一定数量的垃圾生成，如果一个值被传入一个值类型接受者的方法，一个栈上的拷贝会替代在堆上分配内存(但不是保证一定成功)，所以在没搞明白代码想干什么之前，别因为这个原因而选择值类型接受者。

## 何时使用指针类型

- 如果方法需要修改接受者，接受者必须是指针类型。
- 如果接受者是一个包含了 sync.Mutex 或者类似同步字段的结构体，接受者必须是指针，这样可以避免拷贝。
- 如果接受者是一个大的结构体或者数组，那么指针类型接受者更有效率。(多大算大呢？假设把接受者的所有元素作为参数传给方法，如果你觉得参数有点多，那么它就是大)。
- 从此方法中并发的调用函数和方法时，接受者可以被修改吗？一个值类型的接受者当方法调用时会创建一份拷贝，所以外部的修改不能作用到这个接受者上。如果修改必须被原始的接受者可见，那么接受者必须是指针类型。
- 如果接受者是一个结构体，数组或者 slice，它们中任意一个元素是指针类型而且可能被修改，建议使用指针类型接受者，这样会增加程序的可读性

当你看完这个还是有疑虑，还是不知道该使用哪种接受者，那么记住使用指针接受者。

## 关于接受者的命名

社区约定的接受者命名是类型的一个或两个字母的缩写(像 c 或者 cl 对于 Client)。不要使用泛指的名字像是 me，this 或者 self，也不要使用过度描述的名字，最后，如果你在一个地方使用了 c，那么就不要在别的地方使用 cl。


# Go 语言中的方法，接口和嵌入类型

> 来自 <http://studygolang.com/articles/1113>

## 概述

在 Go 语言中，如果一个结构体和一个嵌入字段同时实现了相同的接口会发生什么呢？我们猜一下，可能有两个问题：

- 编译器会因为我们同时有两个接口实现而报错吗？
- 如果编译器接受这样的定义，那么当接口调用时编译器要怎么确定该使用哪个实现？

在写了一些测试代码并认真深入的读了一下标准之后，我发现了一些有意思的东西，而且觉得很有必要分享出来，那么让我们先从 Go 语言中的方法开始说起。

## 方法

Go 语言中同时有函数和方法。一个方法就是一个包含了接受者的函数，接受者可以是命名类型或者结构体类型的一个值或者是一个指针。所有给定类型的方法属于该类型的方法集。

下面定义一个结构体类型和该类型的一个方法：

```go
type User struct {
  Name  string
  Email string
}

func (u User) Notify() error 
```

首先我们定义了一个叫做 User 的结构体类型，然后定义了一个该类型的方法叫做 Notify，该方法的接受者是一个 User 类型的值。要调用 Notify 方法我们需要一个 User 类型的值或者指针：

```go
// User 类型的值可以调用接受者是值的方法
damon := User{"AriesDevil", "ariesdevil@xxoo.com"}
damon.Notify()

// User 类型的指针同样可以调用接受者是值的方法
alimon := &User{"A-limon", "alimon@ooxx.com"}
alimon.Notify() 
```

在这个例子中当我们使用指针时，Go 调整和解引用指针使得调用可以被执行。注意，当接受者不是一个指针时，该方法操作对应接受者的值的副本(意思就是即使你使用了指针调用函数，但是函数的接受者是值类型，所以函数内部操作还是对副本的操作，而不是指针操作 **--意思是说如果在Notify()中修改了Name的值，damon和alimon的Name值是不会变的**)。

我们可以修改 Notify 方法，让它的接受者使用指针类型：

	func (u *User) Notify() error

再来一次之前的调用(注意：当接受者是指针时，即使用值类型调用那么函数内部也是对指针的操作)：

```go
// User 类型的值可以调用接受者是指针的方法
damon := User{"AriesDevil", "ariesdevil@xxoo.com"}
damon.Notify()

// User 类型的指针同样可以调用接受者是指针的方法
alimon := &User{"A-limon", "alimon@ooxx.com"}
alimon.Notify() 
```

**如果Notify()中修改了Name的值，那么damon和alimon的Name值都是会变的。**

如果你不清楚到底什么时候该使用值，什么时候该使用指针作为接受者，你可以去看一下这篇介绍。这篇文章同时还包含了社区约定的接受者该如何命名。

## 接口

Go 语言中的接口很特别，而且提供了难以置信的一系列灵活性和抽象性。它们指定一个特定类型的值和指针表现为特定的方式。从语言角度看，接口是一种类型，它指定一个方法集，所有方法为接口类型就被认为是该接口。

下面定义一个接口：

```go
type Notifier interface {
  Notify() error
} 
```

我们定义了一个叫做 Notifier 的接口并包含一个 Notify 方法。当一个接口只包含一个方法时，按照 Go 语言的约定命名该接口时添加 -er 后缀。这个约定很有用，特别是接口和方法具有相同名字和意义的时候。
我们可以在接口中定义尽可能多的方法，不过在 Go 语言标准库中，你很难找到一个接口包含两个以上的方法。

## 实现接口

当涉及到我们该怎么让我们的类型实现接口时，Go 语言是特别的一个。Go 语言不需要我们显式的实现类型的接口。如果一个接口里的所有方法都被我们的类型实现了，那么我们就说该类型实现了该接口。

让我们继续之前的例子，定义一个函数来接受任意一个实现了接口 Notifier 的类型的值或者指针：

```go
func SendNotification(notify Notifier) error {
  return notify.Notify()
} 
```

SendNotification 函数调用 Notify 方法，这个方法被传入函数的一个值或者指针实现。这样一来一个函数就可以被用来执行任意一个实现了该接口的**值或者指针**的指定的行为。

用我们的 User 类型来实现该接口并且传入一个 User 类型的值来调用 SendNotification 方法：

```go
func (u *User) Notify() error {
  log.Printf("User: Sending User Email To %s<%s>\n",
      u.Name,
      u.Email)
  return nil
}

func main() {
  user := User{
    Name:  "AriesDevil",
    Email: "ariesdevil@xxoo.com",
  }

  SendNotification(user)
}

// Output:
cannot use user (type User) as type Notifier in function argument:
User does not implement Notifier (Notify method has pointer receiver) 
```

为什么编译器不考虑我们的值是实现该接口的类型？接口的调用规则是建立在这些方法的接受者和接口如何被调用的基础上。下面的是语言规范里定义的规则，这些规则用来说明是否我们一个类型的值或者指针实现了该接口：

	类型 *T 的可调用方法集包含接受者为 *T 或 T 的所有方法集

这条规则说的是如果我们用来调用特定**接口方法**的接口变量是一个指针类型，那么方法的接受者可以是值类型也可以是指针类型。显然我们的例子不符合该规则，因为我们传入 SendNotification 函数的接口变量是一个值类型。

	类型 T 的可调用方法集包含接受者为 T 的所有方法

这条规则说的是如果我们用来调用特定接口方法的接口变量是一个值类型，那么方法的接受者必须也是值类型该方法才可以被调用。显然我们的例子也不符合这条规则，因为我们 Notify 方法的接受者是一个指针类型。

语言规范里只有这两条规则，我通过这两条规则得出了符合我们例子的规则：

	类型 T 的可调用方法集不包含接受者为 *T 的方法

我们碰巧赶上了我推断出的这条规则，所以编译器会报错。Notify 方法使用指针类型作为接受者而我们却通过值类型来调用该方法。解决办法也很简单，我们只需要传入 User 值的地址到 SendNotification 函数就好了：

```go
func main() {
  user := &User{
    Name:  "AriesDevil",
    Email: "ariesdevil@xxoo.com",
  }

  SendNotification(user)
}

// Output:
User: Sending User Email To AriesDevil<ariesdevil@xxoo.com>
```

## 嵌入类型

结构体类型可以包含匿名或者嵌入字段。也叫做嵌入一个类型。当我们嵌入一个类型到结构体中时，该类型的名字充当了嵌入字段的字段名。

下面定义一个新的类型然后把我们的 User 类型嵌入进去：

```go
type Admin struct {
  User
  Level  string
}
```

我们定义了一个新类型 Admin 然后把 User 类型嵌入进去，注意这个不叫继承而叫组合。 User 类型跟 Admin 类型没有关系。

我们来改变一下 main 函数，创建一个 Admin 类型的变量并把变量的地址传入 SendNotification 函数中：

```go
func main() {
  admin := &Admin{
    User: User{
      Name:  "AriesDevil",
      Email: "ariesdevil@xxoo.com",
    },
    Level: "master",
  }

  SendNotification(admin)
}

// Output
User: Sending User Email To AriesDevil<ariesdevil@xxoo.com>
```

事实证明，我们可以 Admin 类型的一个指针来调用 SendNotification 函数。现在 Admin 类型也通过来自嵌入的 User 类型的方法提升实现了该接口。

如果 Admin 类型包含了 User 类型的字段和方法，那么它们在结构体中的关系是怎么样的呢？

	当我们嵌入一个类型，这个类型的方法就变成了外部类型的方法，但是当它被调用时，方法的接受者是内部类型(嵌入类型)，而非外部类型。– Effective Go

因此嵌入类型的名字充当着字段名，同时嵌入类型作为内部类型存在，我们可以使用下面的调用方法：

```go
admin.User.Notify()

// Output
User: Sending User Email To AriesDevil<ariesdevil@xxoo.com>
```

这儿我们通过类型名称来访问内部类型的字段和方法。然而，这些字段和方法也同样被提升到了外部类型：

```go
admin.Notify()

// Output
User: Sending User Email To AriesDevil<ariesdevil@xxoo.com>
```

所以通过外部类型来调用 Notify 方法，本质上是内部类型的方法。

下面是 Go 语言中内部类型方法集提升的规则：

------

给定一个结构体类型 S 和一个命名为 T 的类型，方法提升像下面规定的这样被包含在结构体方法集中：

	如果 S 包含一个匿名字段 T，S 和 *S 的方法集都包含接受者为 T 的方法提升。

这条规则说的是当我们嵌入一个类型，嵌入类型的接受者为值类型的方法将被提升，可以被外部类型的值和指针调用。

---------------------------

	对于 *S 类型的方法集包含接受者为 *T 的方法提升

这条规则说的是当我们嵌入一个类型，可以被外部类型的指针调用的方法集只有嵌入类型的接受者为指针类型的方法集，也就是说，当外部类型使用指针调用内部类型的方法时，只有接受者为指针类型的内部类型方法集将被提升。

---------------------------

	如果 S 包含一个匿名字段 *T，S 和 *S 的方法集都包含接受者为 T 或者 *T 的方法提升

这条规则说的是当我们嵌入一个类型的指针，嵌入类型的接受者为值类型或指针类型的方法将被提升，可以被外部类型的值或者指针调用。

---------------------------

这就是语言规范里方法提升中仅有的三条规则，我根据这个推导出一条规则：

	如果 S 包含一个匿名字段 T，S 的方法集不包含接受者为 *T 的方法提升。

这条规则说的是当我们嵌入一个类型，嵌入类型的接受者为指针的方法将不能被外部类型的值访问。这也是跟我们上面陈述的接口规则一致。

## 回答开头的问题

现在我们可以写程序来回答开头提出的两个问题了，首先我们让 Admin 类型实现 Notifier 接口：

```go
func (a *Admin) Notify() error {
  log.Printf("Admin: Sending Admin Email To %s<%s>\n",
      a.Name,
      a.Email)

  return nil
}
```

Admin 类型实现的接口显示一条 admin 方面的信息。当我们使用 Admin 类型的指针去调用函数 SendNotification 时，这将帮助我们确定到底是哪个接口实现被调用了。

现在创建一个 Admin 类型的值并把它的地址传入 SendNotification 函数，来看看发生了什么：

```go
func main() {
  admin := &Admin{
    User: User{
      Name:  "AriesDevil",
      Email: "ariesdevil@xxoo.com",
    },
    Level: "master",
  }

  SendNotification(admin)
}

// Output
Admin: Sending Admin Email To AriesDevil<ariesdevil@xxoo.com>
```

预料之中，Admin 类型的接口实现被 SendNotification 函数调用。现在我们用外部类型来调用 Notify 方法会发生什么呢：

```go
admin.Notify()

// Output
Admin: Sending Admin Email To AriesDevil<ariesdevil@xxoo.com>
```

我们得到了 Admin 类型的接口实现的输出。User 类型的接口实现不被提升到外部类型了。

现在我们有了足够的依据来回答问题了：

- 编译器会因为我们同时有两个接口实现而报错吗？

不会，因为当我们使用嵌入类型时，类型名充当了字段名。嵌入类型作为结构体的内部类型包含了自己的字段和方法，且具有唯一的名字。所以我们可以有同一接口的内部实现和外部实现。

- 如果编译器接受这样的定义，那么当接口调用时编译器要怎么确定该使用哪个实现？

如果外部类型包含了符合要求的接口实现，它将会被使用。否则，通过方法提升，任何内部类型的接口实现可以直接被外部类型使用。

## 总结

在 Go 语言中，方法，接口和嵌入类型一起工作方式是独一无二的。这些特性可以帮助我们像面向对象那样组织结构然后达到同样的目的，并且没有其它复杂的东西。用本文中谈到的语言特色，我们可以以极少的代码来构建抽象和可伸缩性的框架。

# 接口的类型和值

接口被赋值时有类型和值的区别，如果一个接口被赋值为nil，则类型和值都是nil，如果先声明一个结构体变量，该变量的值是nil，把这个变量赋值给接口时，接口的类型不是nil，但值是nil。


下面先声明一个接口和类型：

```go
type intf interface {
	String() string
}

type A struct {
	Name string
}

func (a *A) String() string {
	// 每次都判断一下a是不是nil比较好
	//	if a == nil {
	//		return ""
	//	}
	return a.Name
}
```

有一个打印方法：

```go
func printIntf(i intf) {
	if i == nil {
		fmt.Println("nil")
		return
	}
	fmt.Println(i.String())
}
```

main()方法：

```go
func main() {
	var a3 *A = nil
	printIntf(a3)
}
```

运行时报错：

	invalid memory address or nil pointer dereference

可以看到 `if i == nil` 的判断没有起作用。这是因为此时的i的类型是有值的，但实际的值却是nil，这是 `var a3 *A = nil` 造成的。

一个比较好的解决方案是在String()中判断。

但是如果想在printIntf()中拦截i是nil的情况怎么办？

现在能想到的办法是利用反射：

```go
func printIntf(i intf) {
	// 直接判断i是不是nil适用于printIntf(nil)这种情况
	if i == nil {
		fmt.Println("nil")
		return
	}

	// 可以判断main()中的情况
	v := reflect.ValueOf(i)
	if v.IsNil() {
		fmt.Println("nil - 2")
		return
	}

	fmt.Println(i.String())
}
```

反射有可能影响效率，所以还是尽量少用。遇到这种情况还是重新审视一下自己的设计。



# 方法接收者是值/引用的一个小例子

```go
package main

import "fmt"


func main() {
	a1 := A{"1"}
	a1.F1("11")
	fmt.Println(a1) // {1}
	a1.F2("12")
	fmt.Println(a1) // {12}, (&a1).F2("12")

	a2 := &A{"2"}
	a2.F1("21")
	fmt.Println(a2) // &{2}, (*a2).F1("21"), 修改的是值
	a2.F2("22")
	fmt.Println(a2) // &{22}

	//var _ I = A{} // error，没有实现F2
	var _ I = (*A)(nil)
}

type A struct {
	name string
}

func (a A) F1(s string) {
	a.name = s
}

func (a *A) F2(s string) {
	a.name = s
}

type I interface {
	F1(string)
	F2(string)
}

```

可以这样理解：

- 不涉及接口的情况下：值只有接收者是值类型的方法，同理指针只有接收者是指针的方法，但是Go会自动转换类型。
- 涉及接口的情况下：值变量只拥有接收者是值类型的方法，指针都拥有。