<!-- TOC -->

- [Go Range Loop Internals](#go-range-loop-internals)
    - [Step 1: 请阅读该死的使用手册](#step-1-请阅读该死的使用手册)
        - [range变量](#range变量)
        - [range 表达式](#range-表达式)
    - [Step 2: range支持的数据类型](#step-2-range支持的数据类型)
    - [Step 3: Go编译器源码](#step-3-go编译器源码)
    - [我们了解的](#我们了解的)
    - [附：maps](#附maps)
    - [参考](#参考)

<!-- /TOC -->


# Go Range Loop Internals

整理自 <https://garbagecollected.org/2017/02/22/go-range-loop-internals/>

下面这段程序会终止吗？

```go
	v := []int{1, 2, 3}
	for i := range v {
		v = append(v, i)
	}
```



## Step 1: 请阅读该死的使用手册

第一件事就是去读关于 `range loop` 的文档。文档在 [the for statement section](https://golang.org/ref/spec#For_statements) "For statements with `range` clause" 下。

先来个示例：

```go
for i := range a {
    fmt.Println(i)
}
```

### range变量

`range` 左边变量（上面的`i`）的赋值大部分使用下面两种形式：

- 赋值（`=`）
- 短变量声明（`:=`）

你也可以忽略它。

如果你使用 `:=` ，Go在每次迭代时都会复用这个变量（仅在循环内部）。

### range 表达式

`range` 右边的（上面的 `a`）你可以叫做 `range` 表达式。可以是下面几种：

- 数组 `array`
- 数组的指针
- 切片 `slice`
- 字符串 `string`
- 字典 `map`
- 可以接收值的 `channel`，比如 `chan int` or `chan<- int`

**在循环前 `range` 表达式仅被求值一次**。关于这条规则有一点需要注意：如果你遍历的是数组（或者数组的指针），而你仅仅获取索引，那么只有 `len(a)` 被计算。仅计算`len(a)`意味着表达式`a`可能在编译期就求值了，然后被编译器用一个常量替换。[The spec for the `len` function](https://golang.org/ref/spec#Length_and_capacity) 里解释道：

> 如果 `s` 的类型是数组或者数组的指针，并且`s`不包含`channel`接收或者（非常量）函数调用，那么`len(s)`和`cap(s)`的值是常量值，这种情况下`s`不会被求值。

你怎样才能调用一个表达式仅一次？通过把它赋值给一个变量。

有趣的是说明文档提到了一些关于增/删`map`的（没有提到切片）：

> 如果迭代期间删除了尚未到达的map条目，那么就不会产生相应的迭代值。如果迭代期间创建了map条目，该条目可能在迭代期间产生，也可能被跳过。

我稍后会说到map。

## Step 2: range支持的数据类型

记住一点：**在Go中，你赋值的一切都会拷贝**。如果你赋值一个指针，你会拷贝指针，如果你赋值结构体，你也会拷贝结构体。把参数传给函数也是这样。

| 类型      | 对应的语法糖                |
| ------- | --------------------- |
| 数组      | 就是数组                  |
| 字符串     | 保存有长度字段和底层数组指针的结构体    |
| 切片      | 保存有长度、容量字段和底层数组指针的结构体 |
| 字典      | 一个结构体指针               |
| channel | 一个结构体指针               |

请看博客下方了解这些数据类型的内部结构。

这是什么意思呢？这些例子高亮显示了一些不同。

```go
// copies the entire array
var a [10]int
acopy := a 

// copies the slice header struct only, NOT the backing array
s := make([]int, 10)
scopy := s

// copies the map pointer only
m := make(map[string]int)
mcopy := m
```

所以，如果在 `range` 表达式开始你把一个数组赋值给一个变量（确保它只被求值一次），你将会拷贝整个数组。

## Step 3: Go编译器源码

懒惰的我简单的google了下Go编译器源码。我第一个找的是编译器的GCC版本。有趣的是下面的注释（在`statements.cc `中）：

```go
// Arrange to do a loop appropriate for the type.  We will produce
//   for INIT ; COND ; POST {
//           ITER_INIT
//           INDEX = INDEX_TEMP
//           VALUE = VALUE_TEMP // If there is a value
//           original statements
//   }
```

现在我们已经取得了一些进展。毫不意外地，`range`循环只是内部C风格循环的语法糖。range支持的每种类型都有特定的语法糖。比如，数组：

```go
// The loop we generate:
//   len_temp := len(range)
//   range_temp := range
//   for index_temp = 0; index_temp < len_temp; index_temp++ {
//           value_temp = range_temp[index_temp]
//           index = index_temp
//           value = value_temp
//           original body
//   }
```

切片：

```go
//   for_temp := range
//   len_temp := len(for_temp)
//   for index_temp = 0; index_temp < len_temp; index_temp++ {
//           value_temp = for_temp[index_temp]
//           index = index_temp
//           value = value_temp
//           original body
//   }
```

共同的主题是：

- 所有的一切都只是C风格的循环。
- 你迭代的东西被赋值给一个临时变量。

这是在GCC前端。我知道的大多数人使用gc编译器作为Go的发布。看起来编译器做了差不多相同的事情。

## 我们了解的

1. 循环变量是复用的并且每次迭代都被赋值。
2. range表达式在循环开始前被求值一次，并赋值给一个变量。
3. 迭代map时你可以删除或者添加值。添加的值可能会也可能不会出现在循环中。

掌握了这些后，让我们回过头来看看博客开始处的例子。

```go
	v := []int{1, 2, 3}
	for i := range v {
		v = append(v, i)
	}
```

程序会终止的原因就像下面转换过的代码展示的那样：

```go
for_temp := v
len_temp := len(for_temp)
for index_temp = 0; index_temp < len_temp; index_temp++ {
        value_temp = for_temp[index_temp]
        index = index_temp
        value = value_temp
        v = append(v, index)
}
```

我们知道切片就是个语法糖，它是一个含有指向底层数组指针的结构体。循环在`for_temp`上迭代，`for_temp`是`v`结构体的一个拷贝。变量`v`的任何改变都不会影响另一个结构体拷贝。结构体共享的是底层数组的指针，所以像`v[i] = 1`这样的代码是可以正常工作的。

再一次，像上面例子展示的那样，数组会在循环开始之前被赋值给一个临时变量，这意味着将会拷贝整个数组。指针可以正常工作的原因是拷贝的是指针值而不是数组。

## 附：maps

在说明文档中，我们看到：

- 在迭代字典时添加或者删除元素是安全的。
- 如果你添加了一个元素，这个元素可能会也可能不会出现在下次迭代中。

为什么会是这样？首先我们知道，map是一个结构体的指针。在开始之前，拷贝的是指针而不是内部的数据结构，因此在循环内增删key是可以的。这是有道理的。

为什么你在接下来的迭代中可能看不到你新加的元素？如果你知道hash表是怎么工作的（map实际上就是hash表），你应该知道在hash表内条目的顺序是不固定的。你新加的条目有可能被hash到0索引的位置。所以如果你假设Go会以任意顺序遍历数组，那么你是否会在循环内看到你新加的元素是无法预测的。毕竟你可能已经经过了0索引的位置。在Go map中是不确定会发生什么的，还是让编译器决定吧。



## 参考

1. [The Go Programming Language Specification](https://golang.org/ref/spec)
2. [Go slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)
3. [Go Data Structures](https://research.swtch.com/godata)
4. Inside the map implementation: [slides](https://docs.google.com/presentation/d/1CxamWsvHReswNZc7N2HMV7WPFqS8pvlPVZcDegdC_T4/edit#slide=id.g153a5e64a5_1_0) | [video](https://www.youtube.com/watch?v=Tl7mi9QmLns)
5. Understanding nil: [slides](https://speakerdeck.com/campoy/understanding-nil) | [video](https://www.youtube.com/watch?v=ynoY2xz-F8s)
6. [string source code](https://golang.org/src/runtime/string.go)
7. [slice source code](https://golang.org/src/runtime/slice.go)
8. [map source code](https://golang.org/src/runtime/hashmap.go)
9. [channel source code](https://golang.org/src/runtime/chan.go)

