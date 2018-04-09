<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [变量](#变量)
	- [变量绑定](#变量绑定)
	- [变量与常量](#变量与常量)
	- [隐藏(shadowing)](#隐藏shadowing)
	- [原生类型](#原生类型)
		- [类型列表](#类型列表)
		- [整型](#整型)
		- [浮点型](#浮点型)
		- [字符型](#字符型)
	- [作用域](#作用域)
- [数组](#数组)
	- [基础](#基础)
	- [切片](#切片)
- [元组](#元组)
	- [创建](#创建)
	- [访问](#访问)
- [vectors](#vectors)
	- [创建](#创建)
	- [访问](#访问)
	- [遍历](#遍历)
- [函数](#函数)
	- [声明](#声明)
	- [函数指针](#函数指针)
- [结构体](#结构体)
	- [元组结构体](#元组结构体)
	- [类单元结构体（没有字段）](#类单元结构体没有字段)
	- [枚举](#枚举)
- [分支](#分支)
	- [if](#if)
	- [match](#match)
- [循环](#循环)
	- [loop](#loop)
	- [while](#while)
	- [for](#for)
		- [enumerate()](#enumerate)

<!-- /TOC -->

# 变量

```rust
let x = 5;
let x: i32 = 5; // 指定类型
let (x, y) = (1, 2); // 模式
```


## 变量绑定

`let` 声明一个绑定, 像这样: `let x = 5;`

`let` 的左侧是一个模式, 不仅仅是一个变量, 可以这样写: `let (x, y) = (1, 2);`

绑定默认是不可变的(immutable), 所以 `let x = 5;` 之后不能在赋值 `x = 6;`.

如果想让绑定是可变的, 可以使用 `mut`: `let mut x = 5; x = 6;`

绑定在使用之前必须初始化, 像 `let x: i32;` 就属于未初始化的.


## 变量与常量

```rust
const MAX_POINTS: u32 = 100_000;
```

- 不能对常量使用 mut. 常量总是不可变的.
- 关键字是 const, 并且必须标注类型.
- 常量只能用于常量表达式，而不能作为函数调用的结果，或任何其他只在运行时计算的值。
- Rust 的常量使用下划线分隔的大写字母的命名规范.


## 隐藏(shadowing)

```rust
let x = 5;
let x = x + 1;
let x = x * 2;
```

- 注意不能缺少 let 关键字.
- 隐藏是创建了一个新的变量, 只是重用了之前的变量名. 所以**可以改变变量的类型**.


## 原生类型

rust 可以进行类型推断, 比如 `let x = 5;`, rust 可以推断出 x 的类型为 `i32`.

也可以显式声明类型, 比如 `let x: i32 = 5;`

在编译时就必须知道所有变量的类型.


### 类型列表

- 有符号整型: `i8`, `i16`, `i32`, `i64`.
- 无符号整型: `u8`, `u16`, `u32`, `u64`.
- 可变大小整型：`isize`, `usize`, 依赖底层机器指针大小(64位上就是64bit)。
- 浮点数：`f32`, `f64`  `let x: f64 = 3.14;`
- `bool` : `true` | `false`
- `char` : `let c: char = '中';` `char` 代表unicode字符，占4个字节。
- 数组：定长相同类型 `let arr: [i32; 3] = [1, 2, 3];`
- 切片
- 字符串
- 元组
- vector: `Vec<T>`
- 结构体 `struct`

示例:

```rust
// bool
let x = true;
let y: bool = false;

// char
let c = 'a';
let d = '中';
let e: char = 'e';
```


### 整型

字面值:

| 字面值        | 例子        |
| ------------- | ----------- |
| Decimal       | 98_222      |
| Hex           | 0xff        |
| Octal         | 0o77        |
| Binary        | 0b1111_0000 |
| Byte(u8 only) | b'A'        |

- 除 byte 以外, 可以使用类型后缀, 比如 `57u8`.
- 可以使用 `_` 作为分隔符方便读数, 比如 `1_000`.
- i32 是默认类型, 它通常是最快的, 即使在 64 位架构上.


### 浮点型

```rust
let x = 2.0; // f64
let y: f32 = 3.0; // f32
```

- 两种: `f32` 和 `f64`, 默认是 f64, 因为在现代 CPU 中它与 f32 速度几乎一样，不过精度更高.


### 字符型

```rust
let c = 'z';
let c = 'ℤ';
let c = '😻';
```

- 使用单引号 `''`, 表示 unicode 标量值.



## 作用域

变量绑定只能在被定义的块中存在, 块被 `{}` 包围.

变量可以被隐藏, 后面声明的可以覆盖前面的, 比如:

```rust
let x = 5;
{
    println!("{}", x); // 5
    let x = 6;
    println!("{}", x); // 6
}
println!("{}", x); // 5
let x = 7;  // 重新绑定
println!("{}", x); // 7
```

**隐藏允许将一个变量重绑定为不同的类型, 也可以改变绑定的可变性.**

隐藏并不改变和销毁被绑定的值, 这个值会在离开作用域之前继续存在, 但无法访问到.

```rust
// 改变可变性的例子
let mut x: i32 = 1;
x = 2;
let x = x; // x 现在是不可变的了, 值是2.

// 改变类型的例子
let y = 3;
let y = "hello"
```

# String

- 值在堆上而不是栈上。
- 值是可以改变的, 而原生类型不可以改变。

创建 String 类型:

```rust
let mut s = String::from("hello"); // 从字符串字面值来创建 String
s.push_str(", world!"); // 字符串可以改变
println!("{}", s);
```



# 数组

- 数组长度是固定的, 一旦创建就不能再更改长度.
- 数组分配在栈上而不是堆上.
- 越界访问数组会导致 panic.


## 基础

```rust
let arr: [i32; 3] = [0; 3];
let arr = [0; 3];
let arr = [1, 2, 3];
let arr = ["a", "b", "c"];
```

数组的类型是 `[T, N]`, T 是泛型, N 是长度, 编译时常量.

`[0; 3]` 表示初始化一个数组, 长度为 3, 所有元素都为 0.


- 下标访问: `arr[0]`.
- 获取长度: `arr.len()`.


## 切片

```rust
let arr = [0, 1, 2, 3];
let slice = &arr[1..3]; // 包括1不包括3，[..] 表示全部
```

- 切片是数组的引用.
- `&` 表明切片类似于引用. 切片的类型是 `&[T]`.



# 元组


## 创建

```rust
let tup: (i32, &str) = (3, "hello");
let tup = (3, "hello");
let (x, y) = tup; // 解构
println!("{}, {}", x, y);
```

元组是固定大小的有序列表. 元组内的类型可以是不同的.

如果两个元组类型和数量都相同, 则可以相互赋值:

```rust
let mut x = (1, 2); // x: (i32, i32)
let y = (3, 4); // y: (i32, i32)
x = y;
```


## 访问

可以通过解构来获取元组中的字段:

```rust
let (x, y, z) = (1, 2, 3);
```

使用逗号来消除单元素元组和一个括号中的值的歧义:

```rust
(0,); // 单元素元组
(0);  // 值0
```

还可以通过下标来访问元组中的字段:

```rust
let x = (1, 2, 3);
let a = x.0;
let b = x.1;
let c = x.2;
```


# vectors

Vector 是一个动态数组, 实现为标准库类型 `Vec<T>`.

Vector 总是在堆上分配内存.


## 创建

可以使用 `vec!` 宏来创建:

```rust
let v = vec![1, 2, 3];
println!("v[1] = {}", v[1]);

let v = vec![1; 10]; // 10个元素，初始化为1
println!("v[1] = {}", v[1]);
```

有点像使用数组来构造的 vector.

vector 以连续的 T 的数组的形式存储在堆上, 所以需要在编译的时候就知道 T 的大小.
有些类型在编译时不知道大小, 这时就需要保存一个指向该类型的指针, Box 类型正好符合这种情况.


## 访问

使用下标来访问 vector. **下标的类型只能是 usize**

```rust
let v = vec![1; 10]; // 10个元素，初始化为1
let i: usize = 1; // 类型只能是usize
println!("v[1] = {}", v[i]);
```

越界访问会 panic, 这时可以使用 `get()` 或 `get_mut()`, 它们在越界是返回 None:

```rust
let v = vec![1; 10]; // 10个元素，初始化为1
match v.get(11) {
    Some(x) => println!("get {}", x),
    None => println!("nothing")
}
```


## 遍历

```rust
for i in v {
    println!("get {}", i)
}
for i in &v {
    println!("get {}", i)
}
for i in &mut v {
    println!("get {}", i)
}
```




# 函数

- 命名规范: snake case, 字母都是小写并使用下划线分割单词.
- 当函数体中没有返回值时, 默认返回空元组 `()`.


## 声明

```rust
fn main() {
	hello();
	sum(1, 2);
	println!("incr 7 = {}", incr(7));
}

// 无参无返回值
fn hello() {
	println!("hello");
}

// 有参无返回值
fn sum(a: i32, b: i32) {
	println!("sum({}, {}) = {}", a, b, a + b);
}

// 有参有返回值
fn incr(i: i32) -> i32 {
	// return i + 1;
	i + 1 // 还可以这样写，注意末尾不能有分号
}

// 多个返回值
fn f() -> (i32, i32) {
    (1, 2)
}
// 使用:let (a, b) = f();
```

- 必须为函数参数声明类型.
- 返回值使用 `return` 时是一个语句, 不使用 `return` 则是一个表达式, 而表达式后面不加分号.



## 函数指针

就是将函数的指针赋给常量

```rust
fn main() {
	let f: fn(i: i32)->i32 = incr; // lef 常量名: 类型 = 函数指针, 可以不指定类型：let f = incr;
	f(4);
}

fn incr(i: i32) -> i32 {
	// return i + 1;
	i + 1 // 还可以这样写，注意末尾不能有分号
}
```


# 结构体

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 1, y: 2 };
    println!("x = {}, y = {}", p.x, p.y);
}
```

修改字段值：

```rust
let mut p = Point { x: 1, y: 2 };
p.x = 3;
```

修改结构体的字段然后让其不可变：

```rust
	let mut p = Point { x: 1, y: 2 };
    p.x = 3;
    let p = p; // 现在p不再可变了
	// p.y = 4; // error
```

从另一个结构体复制字段的值：

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

fn main() {
    let p1 = Point { x: 1, y: 2, z: 3 };
    let p2 = Point { x: 4, ..p1 }; // 从p1复制y和z的值
    println!("p2.x = {}, p2.y = {}, p2.z = {}", p2.x, p2.y, p2.z);
}
```

## 元组结构体

使用 `()` 而不是 `{}`, 字段没有名称。

```rust
struct Point(i32, i32, i32);

fn main() {
    let p = Point(1, 2, 3);
    let Point(x, y, z) = p; // 这样来获取结构体的字段值
    println!("x = {}, y = {}, z = {}", x, y, z);
}
```

当结构体只有一个字段时比较有用：

```rust
struct Point(i32);

fn main() {
    let p = Point(1);
    let Point(x) = p;
    println!("x = {}", x);
}
```

## 类单元结构体（没有字段）

```rust
struct Point;
let p = Point;
```

## 枚举

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move {
        x: i32,
        y: i32
    },
    Write(String),
}

fn main() {
    // 创建
    let quit: Message = Message::Quit;
    let change_color: Message = Message::ChangeColor(1, 2, 3);
    let m = Message::Move { x: 1, y: 2 };
    let w = Message::Write("hello".to_string());

    // 匹配
    process_message(quit);
    process_message(change_color);
    process_message(m);
    process_message(w);
}

fn process_message(m: Message) {
    match m {
        Message::Quit => println!("quit message"),
        Message::ChangeColor(r, g, b) => println!("change color({}, {}, {})", r, g, b),
        Message::Move { x, y } => println!("move({}, {})", x, y),
        Message::Write(s) => println!("write {}", s),
    };
}
```


# 分支

## if

```rust
let x = 5;
if x == 5 {
    println!("x == 5")
} else if x == 6 {
    println!("x == 6")
} else {
    println!("x == 7")
}
```

在 let 语句中使用 if:

```rust
let x = 5;
let y = if x == 5 { 10 } else { 11 };
println!("{}", y)
```

if 是一个表达式, 它的值是任何被选择的分支的最后一个表达式的值. if 的每个分支的可能的返回值都必须是 **相同的类型**.

## match

```rust
    let x = 3;
    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("other"),
    };
```

match 是一个表达式：

```rust
    let x = 3;
    let y = match x {
        1 => "one",
        2 => "two",
        3 => "three",
        _ => "other",
    };
    println!("{}", y);
```

匹配多个模式：

```rust
	let x = 2;
    let y = match x {
        1 | 2 => "one or two",
        3 => "three",
        _ => "other",
    };
```

解构：

```rust
	let p = Point { x: 45, y: 90 };
    match p {
        Point { x, y } => println!("point({}, {})", x, y),
    }
```

使用其他名字解构：

```rust
	let p = Point { x: 45, y: 90 };
    match p {
        Point { x: x1, y: y1 } => println!("point({}, {})", x1, y1),
    }
```

只解构部分字段：

```rust
	let p = Point { x: 45, y: 90 };
    match p {
        Point { x, .. } => println!("point({} ..)", x),
    }
```



# 循环
- `break` 退出循环.
- `continue` 继续下次循环.


## loop

- 无限循环, 使用 break 退出循环.

```rust
loop {
    print!("loop forever");
}
```

## while

```rust
let mut i = 0;
let mut done = false;
while !done {
    println!("{}", i);
    i += 1;
    if i == 5 { done = true; }
}
```

## for

```rust
for i in 0..5 {
    println!("{}", i); // 打印 0-4
}
```

抽象形式：

```rust
for var in expression {
  code
}
```

标签：

```rust
'outer: for x in 0..10 {
    'inner: for y in 0..10 {
        if x % 2 == 0 { continue 'outer; }
        if y % 2 == 0 { continue 'inner; }
        println!("x: {}, y: {}", x, y)
    }
}
```

### enumerate()

```rust
for (i, v) in (5..10).enumerate() {
    println!("{}: {}", i, v);
}
```

输出：

```
0: 5
1: 6
2: 7
3: 8
4: 9
```
