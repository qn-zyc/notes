<!-- TOC -->

- [类型Type](#类型type)
    - [整型](#整型)
    - [指针](#指针)
    - [指针指向元素的类型](#指针指向元素的类型)
    - [结构体](#结构体)
        - [结构体字段](#结构体字段)
        - [测试指针和非指针引用的结构体获取字段和方法的能力](#测试指针和非指针引用的结构体获取字段和方法的能力)
    - [数组](#数组)
        - [数组元素类型](#数组元素类型)
    - [map](#map)
    - [chan](#chan)
    - [接口](#接口)
    - [获得接口的值的真实类型](#获得接口的值的真实类型)
    - [判断interface{}指向的元素的类型](#判断interface指向的元素的类型)
    - [Kind](#kind)
    - [获取函数名](#获取函数名)
- [值 Value](#值-value)
    - [设置值](#设置值)
        - [可设置性](#可设置性)
    - [设置和读取数组的值](#设置和读取数组的值)
    - [通过指针设置值](#通过指针设置值)
    - [设置map](#设置map)
    - [interface{}](#interface)
- [标签Tag](#标签tag)
- [reflect.Copy()](#reflectcopy)
    - [copy slice:](#copy-slice)
    - [copy array:](#copy-array)
- [结构体的字段](#结构体的字段)
    - [遍历结构体的字段](#遍历结构体的字段)
    - [判断结构体字段是否是导出的](#判断结构体字段是否是导出的)
- [Select](#select)
- [indirect](#indirect)

<!-- /TOC -->



# 类型Type

## 整型

```go
s1 := reflect.TypeOf((int8)(0)).String() // int8
```

等同于下面这个：

```go
var i8 int8 = 0
reflect.TypeOf(i8).String() // int8
```

## 指针

```go
reflect.TypeOf((*int8)(nil)).String() // *int8
```

等同于：

```go
var i8 *int8 = nil
reflect.TypeOf(i8).String() // *int8
```

## 指针指向元素的类型

```go
reflect.TypeOf((*int8)(nil)).Elem().String() // int8
```

## 结构体

```go
st := (*struct {
	a int
})(nil)
reflect.TypeOf(st).String() // *struct { a int }
reflect.TypeOf(st).Elem().String() // struct { a int }
```

### 结构体字段

```go
reflect.TypeOf(st).Elem().Field(0).Type.String() // int

f, found := reflect.TypeOf(st).Elem().FieldByName("a")
if !found {
	fmt.Println("no field 'a'.")
} else {
	fmt.Println(f.Type.String()) // int
}
```

### 测试指针和非指针引用的结构体获取字段和方法的能力

```go
func main() {
	var i1 interface{}
	i1 = &T{"t1"}

	var i2 interface{}
	i2 = T{"t2"}

	v1 := reflect.ValueOf(i1)
	v2 := reflect.ValueOf(i2)

	// 字段
	//	fmt.Println(v1.FieldByName("A")) // err: call of reflect.Value.FieldByName on ptr Value
	fmt.Println(v2.FieldByName("A"))

	// 方法
	fmt.Println(v1.MethodByName("Fun1"))
	fmt.Println(v1.MethodByName("Fun2"))

	fmt.Println(v2.MethodByName("Fun1"))
	fmt.Println(v2.MethodByName("Fun2")) // <invalid Value>
	
	if m := v2.MethodByName("Fun2"); m.IsValid() {
		fmt.Println("v2.Fun2() is valid.")
	} else {
		fmt.Println("v2.Fun2() is invalid") // this one
	}
}

type T struct {
	A string
}

func (t T) Fun1() {

}

func (t *T) Fun2() {

}
```

* 指针不能获取到字段值。
* 非指针不能获取到接受者是指针的方法，但反之却可以。
* 获取到方法后判断一下是不是可用(`IsValid()`)

## 数组

```go
reflect.TypeOf([32]int{}).String() // [32]int
```

### 数组元素类型

```go
reflect.TypeOf([32]int{}).Elem().String() // int
```


## map

```go
maptype := reflect.TypeOf((map[string]*int32)(nil))
maptype.String()        // map[string]*int32
maptype.Key().String()  // string
maptype.Elem().String() // *int32
```


## chan

```go
chantype := reflect.TypeOf((chan<- string)(nil))
chantype.String()        // chan<- string
chantype.Elem().String() // string
```


## 接口

## 获得接口的值的真实类型

```go
var inter struct {
	E interface{}
}
inter.E = 123.456

innerPtrValue := reflect.ValueOf(&inter)
eField := innerPtrValue.Elem().Field(0)
eField.Type() // interface {}

reflect.ValueOf(eField.Interface()).Type() // float64
```

或者

```go
eField.Elem().Type()
```

## 判断interface{}指向的元素的类型

```go
var i interface{}
i = "hello"

t := reflect.ValueOf(i).Type()
fmt.Println(t) // string
```

但如果传给i的是个指针的话：

```go
var i interface{}
s := "hello"
i = &s

t := reflect.ValueOf(i).Type()
fmt.Println(t) // *string
```

如果使用`reflect.ValueOf(i).Elem().Type()`可以获得，但是这个对于不是指针的话就会报错，因为Elem()不能用于string

可以使用reflect.Indirect():

```go
var i interface{}
s := "hello"
i = &s // i = s也是输出string

v := reflect.ValueOf(i)
t := reflect.Indirect(v).Type()
fmt.Println(t) // string
```

下面是Indirect()的实现：

```go
func Indirect(v Value) Value {
	if v.Kind() != Ptr {
		return v
	}
	return v.Elem()
}
```

可以参考一下接口为nil时类型和值都必须是nil的介绍。

## Kind

```go
reflect.TypeOf((int)(0)).Kind() == reflect.Int
reflect.TypeOf((*int)(nil)).Kind() == reflect.Ptr
reflect.TypeOf((map[string]string)(nil)).Kind() == reflect.Map
```

反射对象的 Kind 描述了底层类型，而不是静态类型。如果一个反射对象包含了用户定义的整数类型的值，就像:

```go
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
```

v 的 Kind 仍然是 reflect.Int，尽管 x 的静态类型是 MyInt，而不是 int。换句话说，Kind 无法从 MyInt 中区分 int，而 Type 可以。



## 获取函数名

```go
func getFuncName(f interface{}) string {
	longName := runtime.FuncForPC(reflect.ValueOf(f).Pointer()).Name()
	parts := strings.Split(longName, ".")
	if len(parts) > 0 {
		return parts[len(parts)-1]
	}
	return "?"
}
```





# 值 Value

## 设置值

SetXXX系列函数要求CanSet()返回true。

先判断类型在设置值

```go
i := 123
iptr := &i
v := reflect.ValueOf(iptr).Elem()
if v.Kind() == reflect.Int {
	v.SetInt(321)
}
fmt.Println(i) // 321
```

不判断类型直接设置值

```go
var i int = 123
iptr := &i
v := reflect.ValueOf(iptr).Elem()
v.Set(reflect.ValueOf(int(321)))
fmt.Println(i) // 321
```


### 可设置性

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
```

这里报错是因为 v 无法设置值给 x。

Value的 CanSet 方法提供了值的设置性；在这个例子中，

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:" , v.CanSet()) // false
```

创建 v 时传递的是 x 的副本给 reflect.ValueOf()，而不是 x 本身。


```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // 注意：获取 X 的地址。
fmt.Println("type of p:", p.Type()) // type of p: *float64
fmt.Println("settability of p:" , p.CanSet()) // settability of p: false
```

反射对象 p 并不是可设置的，而且我们也不希望设置 p，实际上是 *p。为了获得 p 指向的内容，调用值上的 Elem 方法，从指针间接指向，然后保存反射值的结果叫做 v：

```go
v := p.Elem()
fmt.Println("settability of v:" , v.CanSet()) // true
v.setFloat(7.1)
```

通过 reflect.Indirect() 方法来取指针指向的具体值更好。



## 设置和读取数组的值

```go
arrval := reflect.ValueOf(&[...]int{1, 2, 3}).Elem()
arrval.Index(1).SetInt(5)

out := "["
for i := 0; i < arrval.Len(); i++ {
	out += strconv.Itoa(int(arrval.Index(i).Int())) + ","
}
out += "]"
fmt.Println(out) // [1,5,3,]
```

ValueOf()的参数需是数组指针，下面这个是错误的

```go
reflect.ValueOf([...]int{1, 2, 3})
```

对于切片来说则不需要这么麻烦(因为通过切片获取到的数组是指针)

```go
arrval := reflect.ValueOf([]int{1, 2, 3})
arrval.Index(1).SetInt(5)
```

## 通过指针设置值

```go
var ip *int32
var i int32 = 123
vip := reflect.ValueOf(&ip)      // **int32
vi := reflect.ValueOf(&i).Elem() // 非得这样吗？(直接传递i给ValueOf会复制i)
vip.Elem().Set(vi.Addr()) // *int32.Set(*int32)
fmt.Println(*ip) // 123
```

将指针设置为零值

```go
var i int32 = 123
ip := &i

vp := reflect.ValueOf(&ip).Elem()
vp.Set(reflect.Zero(vp.Type()))
fmt.Println(ip) // <nil>
```

## 设置map

设置map为空值

```go
m := make(map[string]int)
m["a"], m["b"] = 1, 2

mp := reflect.ValueOf(&m).Elem()
mp.Set(reflect.Zero(mp.Type()))
fmt.Println(m)        // map[]
fmt.Println(m == nil) // true
```

## interface{}

从匿名结构体通过Interface()还原到实际类型

```go
b := struct{ a, b, c, d int64 }{1, 2, 3, 4}
v := reflect.ValueOf(b)
b1 := v.Interface().(struct {
	a, b, c, d int64
})
fmt.Println(b1) // {1 2 3 4}
```

interface()不可用于结构体中未导出字段

```go
var st struct {
	PublicField  string
	privateField string
}

vSt := reflect.ValueOf(&st).Elem()
vSt.Field(0).IsValid() // true
vSt.Field(1).IsValid() // true

defer func() {
	if err := recover(); err != nil {
		fmt.Println("has error :", err)
	}
}()
//vSt.Field(0).Interface() // 没有panic
vSt.Field(1).Interface() // panic
```


# 标签Tag

```go
var st struct {
	f1 string `tag1:"cont1" tag2:cont2 tag3:3`
}
var f1Tag = reflect.TypeOf(st).Field(0).Tag
f1Tag.Get("tag1") // cont1
f1Tag.Get("tag2") // 空字符串
f1Tag.Get("tag3") // 空字符串
```

# reflect.Copy()

## copy slice:
```go
a := []int{1, 2, 3, 4, 5, 6}
b := []int{11, 12, 13, 14, 15}
aa := reflect.ValueOf(&a).Elem()
bb := reflect.ValueOf(&b).Elem()
aa.SetLen(4) // 限制长度只到第4个
reflect.Copy(bb, aa) // 赋值aa的前4个到b里
aa.SetLen(6) // 回复长度
fmt.Println(a) // [1 2 3 4 5 6]
fmt.Println(b) // [1 2 3 4 15]
```

## copy array:
```go
a := [...]int{1, 2, 3}
b := [...]int{11, 12, 13, 14, 15}
aa := reflect.ValueOf(&a).Elem()
bb := reflect.ValueOf(&b).Elem()
reflect.Copy(bb, aa)
fmt.Println(b) // [1 2 3 14 15]
```


# 结构体的字段

## 遍历结构体的字段

```go
func main() {
	t := &T{"a", "b"}
	val := reflect.Indirect(reflect.ValueOf(t))
	typ := val.Type()

	for i := 0; i < val.NumField(); i++ {
		f := typ.Field(i)
		fmt.Println(f.Name)
	}
}

type T struct {
	PublicField  string
	privateField string
}
```

输出：

	PublicField
	privateField


## 判断结构体字段是否是导出的

```go
t := &T{"a", "b"}
val := reflect.Indirect(reflect.ValueOf(t))
typ := val.Type()
typepath := typ.PkgPath()
fmt.Println("type pkgPath: ", typepath) // type pkgPath:  main

for i := 0; i < val.NumField(); i++ {
	f := typ.Field(i)
	p := f.PkgPath
	fmt.Println(f.Name, " => ", p)
}

//	 PublicField  =>
//	 privateField  =>  main
```



# Select

文档: <https://golang.org/pkg/reflect/#Select>

和select结构功能一样，但这个可以直接指定数组。

测试代码:

```go
import (
	"fmt"
	"reflect"
)

func main() {
	selectCase := make([]reflect.SelectCase, 3)

	for i := 0; i < 3; i++ {
		c := make(chan string)

		selectCase[i].Dir = reflect.SelectRecv
		selectCase[i].Chan = reflect.ValueOf(c)

		go func(i int, c chan string) {
			for j := 0; j < 5; j++ {
				c <- fmt.Sprintf("groutine#%d send %d", i, j)
			}
			fmt.Printf("groutine#%d closed\n", i)
			close(c)
		}(i, c)
	}

	done := 0
	finished := 0
	for finished < len(selectCase) {
		chosen, recv, recvOk := reflect.Select(selectCase)
		if recvOk {
			done++
			fmt.Printf("receive #%d, value=%s\n", chosen, recv.String())
		} else {
			fmt.Printf("#%d closed\n", chosen)
			selectCase = append(selectCase[:chosen], selectCase[chosen+1:]...)
		}
	}

	fmt.Println("done:", done) // 15
}
```



# indirect

```go
// From html/template/content.go
// Copyright 2011 The Go Authors. All rights reserved.
// indirect returns the value, after dereferencing as many times
// as necessary to reach the base type (or nil).
func indirect(a interface{}) interface{} {
	if a == nil {
		return nil
	}
	if t := reflect.TypeOf(a); t.Kind() != reflect.Ptr {
		// Avoid creating a reflect.Value if it's not a pointer.
		return a
	}
	v := reflect.ValueOf(a)
	for v.Kind() == reflect.Ptr && !v.IsNil() {
		v = v.Elem()
	}
	return v.Interface()
}
```

