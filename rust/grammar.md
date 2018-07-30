<!-- TOC -->

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
- [String](#string)
    - [创建](#创建)
    - [基本操作](#基本操作)
    - [索引字符串](#索引字符串)
    - [遍历](#遍历)
- [Slices](#slices)
    - [字符串 slice](#字符串-slice)
- [数组](#数组)
    - [基础](#基础)
    - [切片](#切片)
- [元组](#元组)
    - [创建](#创建-1)
    - [访问](#访问)
- [vector](#vector)
    - [创建](#创建-2)
    - [更新](#更新)
    - [访问](#访问-1)
        - [无效引用](#无效引用)
    - [遍历](#遍历-1)
    - [传递给函数](#传递给函数)
- [map](#map)
    - [创建](#创建-3)
    - [访问](#访问-2)
    - [遍历](#遍历-2)
    - [更新](#更新-1)
- [函数 fn](#函数-fn)
    - [声明](#声明)
    - [函数指针](#函数指针)
    - [闭包](#闭包)
        - [在结构体中使用闭包](#在结构体中使用闭包)
        - [闭包会捕获其环境](#闭包会捕获其环境)
- [结构体 struct](#结构体-struct)
    - [创建](#创建-4)
        - [元组结构体(tuple structs)](#元组结构体tuple-structs)
        - [类单元结构体(没有字段)](#类单元结构体没有字段)
    - [方法](#方法)
        - [关联函数](#关联函数)
    - [在结构体中使用 trait](#在结构体中使用-trait)
    - [状态模拟](#状态模拟)
- [枚举](#枚举)
    - [定义](#定义)
    - [在枚举上定义方法](#在枚举上定义方法)
    - [标准库的Option](#标准库的option)
    - [使用 match 匹配枚举](#使用-match-匹配枚举)
- [分支](#分支)
    - [if](#if)
    - [match](#match)
        - [匹配守卫(match guard)](#匹配守卫match-guard)
        - [@绑定](#绑定)
        - [模式匹配](#模式匹配)
            - [使用 ref 和 mut ref 创建引用](#使用-ref-和-mut-ref-创建引用)
        - [if let](#if-let)
- [循环](#循环)
    - [loop](#loop)
    - [while](#while)
    - [for](#for)
        - [enumerate()](#enumerate)
- [迭代器 iterator](#迭代器-iterator)
    - [迭代器的方法](#迭代器的方法)
    - [自定义迭代器](#自定义迭代器)
- [模块 mod](#模块-mod)
    - [定义](#定义-1)
    - [在不同文件中定义模块](#在不同文件中定义模块)
    - [公有和私有](#公有和私有)
    - [使用模块](#使用模块)
        - [使用 use 导入作用域](#使用-use-导入作用域)
        - [使用 super 访问父模块](#使用-super-访问父模块)
        - [使用 pub use 重导出模块](#使用-pub-use-重导出模块)
- [工作空间](#工作空间)
- [错误处理](#错误处理)
    - [触发panic](#触发panic)
    - [Result](#result)
    - [匹配不同的错误](#匹配不同的错误)
    - [unwrap 和 expect](#unwrap-和-expect)
    - [传播错误](#传播错误)
- [trait](#trait)
    - [定义trait](#定义trait)
    - [实现trait](#实现trait)
- [泛型](#泛型)
    - [定义泛型](#定义泛型)
    - [Trait Bounds](#trait-bounds)
    - [不同泛型实现不同的方法](#不同泛型实现不同的方法)
- [测试](#测试)
    - [判断函数](#判断函数)
        - [should_panic](#should_panic)
    - [自定义错误信息](#自定义错误信息)
    - [引用外部函数](#引用外部函数)
    - [运行测试](#运行测试)
    - [测试的组织结构](#测试的组织结构)
        - [单元测试](#单元测试)
        - [集成测试](#集成测试)
- [并发 concurrency](#并发-concurrency)
    - [使用 spawn 创建新线程](#使用-spawn-创建新线程)
    - [使用 join 等待线程结束](#使用-join-等待线程结束)
    - [线程与 move 闭包](#线程与-move-闭包)
    - [使用 channel 在线程间传递数据](#使用-channel-在线程间传递数据)
    - [互斥器 mutex](#互斥器-mutex)
        - [多个线程间使用 mutex 访问数据](#多个线程间使用-mutex-访问数据)

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

## 创建

```rust
let mut s = String::from("hello"); // 从字符串字面值来创建 String
s.push_str(", world!"); // 字符串可以改变
println!("{}", s);

let data = "initial contents";
let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();

// 多行文本
let contents = "\
one line
two line
three line";
```


## 基本操作

追加字符串:

```rust
let mut s = String::from("one");
s.push_str(" two");
let three = " three";
s.push_str(three); // 这里将 three 的所有权转移了, 如果不想转移的话就使用 s.push_str(&three);

s.push('.'); // 仅追加一个字符.
println!("the string is: {}", s)
```

连接字符串:

```rust
let s1 = String::from("hello, ");
let s2 = String::from("world!");
let s = s1 + &s2; // Note that s1 has been moved here and can no longer be used
println!("{}", s)
```

`+` 的签名就像这样: `fn add(self, s: &str) -> String`, 所以 s1 只能是 String, 而 s2 只能是 &str, 这里 `&s2` 的类型是 `&String`, 是因为 Rust 将 `&String` 强转成 `&str` 了. s1 的所有权发生了转移, 所以执行完 `s1 + &s2` 后就不能再使用了.

级联多个字符串:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```


## 索引字符串

```rust
let s1 = String::from("hello");
let h = s1[0];
```

上面的代码会报错, 原因是 **Rust 的字符串不支持索引**. 因为字符串是按 utf-8 编码的, 直接索引获取的值可能不是一个有效的字符.

可以通过字符串 slice 来获取字符串的某个片段:

```rust
let s = String::from("hello");
let s1 = &s[0..1];
println!("{}", s1);

let s = String::from("中文");
let s2 = &s[0..1]; // 这里在运行时将 panic
println!("{}", s2);
```

上面代码在获取无效的字符串 slice 依然会 panic, 所以需要谨慎使用这个操作.


## 遍历

```rust
let s = String::from("中文");
for c in s.chars() {
    println!("{}", c);
}
```

chars() 会将字符串分隔成单独的 unicode 值, 上例输出 `中`, `文` 两个字. 直接遍历字符串: `for c in &s` 会 panic.

下例输出字符串的字节列表:

```rust
let s = String::from("中文");
for c in s.bytes() {
    println!("{}", c);
}

// 228 184 173 230 150 135
```





# Slices

- slice 没有所有权.

## 字符串 slice

```rust
let s = String::from("hello world");
let hello = $s[0..5];
let world = &s[6..11];
```

- hello 和 world 的类型是 `&str`, 是一个不可变引用.
- `[start..end]` 包括 start, 不包括 end.
- start 可以省略, 默认是 0: `&s[..5]`.
- end 也可以省略, 默认是 len: `&s[3..]`.
- start 和 end 都省略: `&s[..]`.

字符串字面值就是 slice: `let s = "hello";`, s 的类型是 `&str`, 指向二进制程序特定位置的 slice.


------------------------------

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

数组的类型是 `[T; N]`, T 是泛型, N 是长度, 编译时常量.

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


# vector

- vector 是一个动态数组, 实现为标准库类型 `Vec<T>`.
- vector 总是在堆上分配内存, 彼此相邻的排列所有的值.


## 创建

```rust
let mut v = Vec::new();
```

如果只是使用上面的代码来创建, 则会报错, 因为 Rust 无法推断出 v 的类型, 所以需要显式声明类型: `let mut v: Vec<i32> = Vec::new();`.

```rust
let mut v = Vec::new();
v.push(1)
```

上面的 v 也没有声明类型, 但是下面的 push(1) 可以让 Rust 推断出 T 的类型是 i32.

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


## 更新

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```


## 访问

```rust
let v = vec![1; 10];

let i: usize = 2;
let a: &i32 = &v[i];
println!("a = {}", a);

let b: Option<&i32> = v.get(i);
match b {
    Some(x) => println!("b = {}", x),
    None => println!("not found"),
}
```

- 使用下标的方式时下标必须是 usize 类型. 如果越界访问, 则会 panic.
- 使用 get() 时返回的是 Option, 如果越界访问, 则返回 None.


### 无效引用

```rust
let mut v = vec![1, 2, 3, 4, 5];
let first = &v[0];
v.push(6);
```

上面的程序会报错, 不能这么做的原因是由于 vector 的工作方式。在 vector 的结尾增加新元素时，在没有足够空间将所有所有元素依次相邻存放的情况下，可能会要求分配新内存并将老的元素拷贝到新的空间中。这时，第一个元素的引用就指向了被释放的内存。借用规则阻止程序陷入这种状况.


## 遍历

```rust
// 获取每个元素的不可变引用并打印
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}

// 获取每个元素的可变引用, 并修改每个元素的值
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
    println!("i = {}", i);
}
```


## 传递给函数

```rust
fn main() {
    let list = vec![34, 50, 25, 100, 65];
    let result = largest(&list);
    println!("The largest number is {}", result);
}

fn largest(list: &[i32]) -> i32 { // 传的是切片
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```



# map

## 创建

```rust
use std::collections::HashMap;

fn main() {
    let mut m = HashMap::new();
    m.insert(String::from("one"), 50);
    m.insert(String::from("two"), 100);
    let three = String::from("three");
    m.insert(three, 111);
}
```

- 键必须是相同的类型, 值也必须是相同的类型.
- 拥有所有权的值(比如上面的 three)一旦放入了 HashMap, 所有权就被转移了. 也可以放入引用, 但要保证引用的有效期比 HashMap 长.

另一个构建哈希 map 的方法是使用一个元组的 vector 的 collect 方法，其中每个元组包含一个键值对。

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

这里 HashMap<_, _> 类型注解是必要的，因为可能 collect 很多不同的数据结构，而除非显式指定否则 Rust 无从得知你需要的类型。但是对于键和值的类型参数来说，可以使用下划线占位，而 Rust 能够根据 vector 中数据的类型推断出 HashMap 所包含的类型。


## 访问

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name); // 返回的是 Option<i32> 类型
```


## 遍历

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```


## 更新

**值不存在时插入, 存在时覆盖**:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

**值不存在时插入, 存在时不做操作**:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

`or_insert`: 如果 key 存在旧返回旧的 entry, 如果不存在就插入值并返回新的 entry.


**计数器**:

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

`or_insert` 返回的是值的一个可变引用(`&mut V`), 所以为了赋值必须使用 `*` 解引用.




------------------------------
# 函数 fn

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


在参数中使用解构:

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```




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


## 闭包

```rust
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

`||` 之间是参数列表, 多个参数使用逗号分隔: `|param1, param2|`.

不需要注明类型, 当然也可以注明类型:

```rust
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 } // 函数定义
let add_one_v2 = |x: u32| -> u32 { x + 1 }; // 标注了类型的闭包
let add_one_v3 = |x|             { x + 1 }; // 省略了类型
let add_one_v4 = |x|               x + 1  ; // 省略了花括号
```

闭包参数的类型是通过调用时推导出来的, 所以调用时不能指定不同的类型:

```Rust
let example_closure = |x| x;

let s = example_closure(String::from("hello")); // String
let n = example_closure(5); // u32, Error!!!
```

### 在结构体中使用闭包

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

`Fn` 系列 trait 由标准库提供。所有的闭包都实现了 trait `Fn`、`FnMut` 或 `FnOnce` 中的一个.

在结构体中的闭包的参数需要指定类型.


### 闭包会捕获其环境

```rust
fn main() {
    let x = 4;
    let equal_to_x = |z| z == x;
    let y = 4;
    assert!(equal_to_x(y));
}
```

而函数不行:

```rust
fn main() {
    let x = 4;
    fn equal_to_x(z: i32) -> bool { z == x } // Error!!!
    let y = 4;
    assert!(equal_to_x(y));
}
```

闭包可以通过三种方式捕获其环境，他们直接对应函数的三种获取参数的方式：获取所有权，不可变借用和可变借用。这三种捕获值的方式被编码为如下三个 Fn trait：

* `FnOnce` 消费从周围作用域捕获的变量，闭包周围的作用域被称为其环境，environment。为了消费捕获到的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。其名称的 Once 部分代表了闭包不能多次获取相同变量的所有权的事实，所以它只能被调用一次。
* `Fn` 从其环境不可变的借用值
* `FnMut` 可变的借用值所以可以改变其环境

如果我们希望强制闭包获取其使用的环境值的所有权，可以在参数列表前使用 `move` 关键字。这个技巧在将闭包传递给新线程以便将数据移动到新线程中时最为实用。

```rust
fn main() {
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x; // x 的所有权被移到闭包内
    println!("can't use x here: {:?}", x); // 这里不能在使用 x 了
    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```




------------------------------
# 结构体 struct

## 创建

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

实例化:

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

实例化并修改字段值:

```rust
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```

- user1 实例必须是可变的.

从函数中返回实例:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

变量与字段同名时的简写形式:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,    // 简写
        username, // 简写
        active: true,
        sign_in_count: 1,
    }
}
```

从其他对象创建对象:

```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1 // !!!
};
```

- `..` 语法指定了剩余未显式设置值的字段应有与给定实例对应字段相同的值。



### 元组结构体(tuple structs)

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
    println!("x = {}", p.0) // 使用 0 也可以访问值
}
```




### 类单元结构体(没有字段)

```rust
struct Point;
let p = Point;
```



## 方法

定义方法:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

- `impl` 是 `implementation` 的缩写.
- `&self` 是代替 `&Rectangle`, 是一个不可变引用. 要获取一个可变引用可以使用 `&mut self`. 通过仅仅使用 self 作为第一个参数来使方法获取实例的所有权是很少见的, 这种技术通常用在当方法将 self 转换成别的实例的时候，这时我们想要防止调用者在转换之后使用原始的实例。
- 自动引用和解引用（automatic referencing and dereferencing）: 当使用 `object.something()` 调用方法时，Rust 会自动增加 `&`、`&mut` 或 `*` 以便使 object 符合方法的签名.
```rust
p1.distance(&p2);
(&p1).distance(&p2); // 与上面等价
```

每个结构体都允许拥有多个 impl 块:

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```


### 关联函数

- 关联函数: 在 impl 块中定义不以 self 作为参数的函数.
- 关联函数经常被用作返回一个结构体新实例的构造函数.

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

调用: `let sq = Rectangle::square(3);`.


## 在结构体中使用 trait

```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<Draw>>, // 直接用 Draw 不能在编译器确定大小
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        println!("draw a Button");
    }
}

pub struct SelectBox {
    pub width: u32,
    pub height: u32,
    pub options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        println!("draw a SelectBox");
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(Button {
                width: 10,
                height: 20,
                label: String::from("OK"),
            }),
            Box::new(SelectBox {
                width: 23,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
        ],
    };

    screen.run();
}
```


## 状态模拟

```rust
trait State {
    // 只对这个类型的 Box 有效. 获取 Box<Self> 的所有权, 使老状态无效, 返回新的状态.
    // 请求审核博文时的状态
    fn request_review(self: Box<Self>) -> Box<State>;
    // 博文审核通过的状态
    fn approve(self: Box<Self>) -> Box<State>;
    // 返回博文内容
    fn content<'a>(&self, _: &'a Post) -> &'a str {
        ""
    }
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<State> {
        Box::new(PendingReview {})
    }
    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }
    fn approve(self: Box<Self>) -> Box<State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }
    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}

pub struct Post {
    state: Option<Box<State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }

    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn content(&self) -> &str {
        // as_ref() 会返回 Option<&Box<State>>
        self.state.as_ref().unwrap().content(&self)
    }

    pub fn request_review(&mut self) {
        // take() 会返回带所有权的 Box<State>, 并且 self.state 为 None.
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }

    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    println!("content: {}", post.content());

    post.request_review();
    println!("content: {}", post.content());

    post.approve();
    println!("content: {}", post.content());
}
```




# 枚举

## 定义

```rust
// 最简单的一种定义
enum IpAddrKind {
    V4,
    V6,
}
fn route(ip_type: IpAddrKind) { }
route(IpAddrKind::V4);
route(IpAddrKind::V6);
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;

// 在枚举值中存储值
enum IpAddr {
    V4(String),
    V6(String),
}
let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));

enum IpAddr {
    V4(u8, u8, u8, u8), // 可以是不同类型的的
    V6(String),
}
let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));

// 存储各种类型
enum Message {
    Quit, // 没有关联任何数据
    Move { x: i32, y: i32 }, // 包含一个匿名结构体
    Write(String), // 包含单独一个 String
    ChangeColor(i32, i32, i32), // 包含三个 i32
}
let quit: Message = Message::Quit;
let change_color: Message = Message::ChangeColor(1, 2, 3);
let m = Message::Move { x: 1, y: 2 };
let w = Message::Write("hello".to_string());
```


## 在枚举上定义方法

```rust
impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```


## 标准库的Option

```rust
enum Option<T> {
    Some(T),
    None,
}
let some_number = Some(5);
let some_string = Some("a string");
let absent_number: Option<i32> = None; // 需要指定 T 的类型, 编译器推断不出来.
```

- 它被包含在了 prelude 之中，这意味着我们不需要显式引入作用域.
- 不需要 `Option::` 前缀来直接使用 `Some` 和 `None`.



## 使用 match 匹配枚举

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

多行代码可以使用 `{}` 括起来:

```rust
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

使用 `_` 和 `()` 忽略分支(match必须匹配所有可能的情况):

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

match 是一个表达式, 可以返回值:

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

使用 `|` 匹配多个条件:

```rust
let x = 2;
let y = match x {
    1 | 2 => "one or two",
    3 => "three",
    _ => "other",
};
```

使用 `...` 匹配范围:

```rust
let x = 5;

match x {
    1 ... 5 => println!("one through five"), // 匹配 1-5
    _ => println!("something else"),
}
```

范围只允许用于数字或 char 值，因为编译器会在编译时检查范围不为空。char 和 数字值是 Rust 唯一知道范围是否为空的类型。

```rust
let x = 'c';

match x {
    'a' ... 'j' => println!("early ASCII letter"),
    'k' ... 'z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```



解构结构体:

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

匹配特定值:

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x), // 当 y = 0 时匹配
        Point { x: 0, y } => println!("On the y axis at {}", y), // 当 x = 0 时匹配
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```



匹配 Option:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```



### 匹配守卫(match guard)

匹配守卫（match guard）是一个指定与 match 分支模式之后的额外 if 条件，它也必须被满足才能选择此分支。匹配守卫用于表达比单独的模式所能允许的更为复杂的情况。

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}


let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"), // 作用于 4,5,6
    _ => println!("no"),

```


### @绑定

at 运算符 @ 允许我们在创建一个存放值的变量的同时测试其值是否匹配模式。

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3...7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10...12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

第一个分支限定 id 的值在 3-7, 并且将值绑定到 `id_variable`, 这样在 println 中就可以使用 `id_variable` 了.

第二个分支限定 id 的值在 10-12, 但没有绑定变量, 所以在 println 不能使用 id 的值.

第三个分支使用了结构体字段简写, 没有限定范围.






### 模式匹配


#### 使用 ref 和 mut ref 创建引用

```rust
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

上例将编译失败, 因为 `robot_name` 的部分所有权被转移到 name 中了, 所以 println 就获取不到所有权了.

```rust
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

使用 `ref name`, name 只是获取了引用而没有获取所有权.

```rust
let mut robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref mut name) => *name = String::from("Another name"),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

使用 `ref mut name` 创建了一个可变引用.






### if let

处理只匹配一个模式的值而忽略其他模式的情况, 如下:

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

可以使用下面的简写形式:

```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```

还可以使用 else 来代替 `_` 分支的代码:

```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```

上面的代码可以使用下面的简写形式:

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

更复杂些的例子:

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

不能将模式匹配和判断组合为 `if let Ok(age) = age && age > 30`, 因为 age 直到大括号开始时才生效.

if let 表达式的缺点在于其穷尽性没有为编译器所检查，而 match 表达式则检查了。



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

`while let` 只要模式匹配就一直进行循环. 下例中展示打印栈中的所有值:

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
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



------------------------------
# 迭代器 iterator

```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
for val in v1_iter { // for 获取了 v1_iter 的所有权, 并在后台使 v1_iter 可变.
    println!("Got: {}", val);
}
```

迭代器实现了标准库中的 `Iterator` trait.

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter(); // 这里需要可变, 因为 next() 改变了内部状态

    assert_eq!(v1_iter.next(), Some(&1)); // next() 返回了 vec 中值的不可变引用.
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

`iter` 方法生成一个不可变引用的迭代器。如果我们需要一个获取 v1 所有权并返回拥有所有权的迭代器，则可以调用 `into_iter` 而不是 `iter`。类似的，如果我们希望迭代可变引用，则可以调用 `iter_mut` 而不是 `iter`。


## 迭代器的方法

求和:

```rust
fn main() {
    let v = vec![1, 2, 3];
    let iter = v.iter();
    let total: i32 = iter.sum();
    println!("total is {}", total)
}
```

使用 map 产出另一个迭代器:

```rust
fn main() {
    let v = vec![1, 2, 3];
    let v1: Vec<_> = v.iter().map(|x| x + 1).collect();
    println!("{:?}", v1) // [2, 3, 4]
}
```


## 自定义迭代器

实现一个只产生 1-5 的计数器:

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let c = Counter::new();
    for n in c {
        println!("{}", n)
    }
}
```





------------------------------
# 模块 mod
- 使用 `cargo new --lib mod_name` 可以创建一个库.
- 使用 `cargo build` 来编译.
- 模块的入口点是 `lib.rs` 文件.
- 子模块的入口是 `mod.rs` 文件.

## 定义

在 `src/lib.rs`(只能叫 lib.rs, 固定会去找这个文件) 中定义一个 `network` 模块:

```rust
mod network {
    fn connect(){
    }
}
```

调用模块中的方法使用 `network::connect()`.

**在同一个文件中定义多个模块**:

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

调用方法: `network::connect()`, `client::connect()`.


**在模块中定义子模块**:

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

调用方法是: `network::client::connect()`.


## 在不同文件中定义模块

如果是在同一个文件中定义所有模块的话, 看起来是这样的:

```rust
// src/lib.rs
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

先将 client 移出去

```rust
// src/lib.rs
mod client; // 只声明, 没有实现

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}

// src/client.rs
fn connect() {
}
```

client.rs 中不需要再用 mod 声明了, 因为已经在 lib.rs 中声明过了.

再将 network 移出去:

```rust
// src/lib.rs
mod client;

mod network;

// src/network.rs
fn connect() {
}

mod server {
    fn connect() {
    }
}
```


如果想将子模块 server 也移到单独的一个文件中, 需要先创建一个 network 目录, 在这个目录中创建 server.rs:

```rust
// src/lib.rs
mod network;

// src/network/mod.rs 只能叫 mod.rs
fn connect() {}

mod server;

// src/network/server.rs
fn connect() {}
```


## 公有和私有

- 模块, 函数等默认是私有的, 需要设置为公有需要添加 `pub`.

```rust
pub mod network;

pub fn hello(){}
```


## 使用模块

调用外部模块:

```rust
extern crate communicator;

fn main() {
    communicator::hello();
    communicator::network::connect();
}
```

调用内部模块:

```rust
mod net {
    pub fn connect() {
        println!("net.connect()")
    }
}

fn main() {
    net::connect()
}
```


### 使用 use 导入作用域

- use 是相对于根模块的.


```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules(); // 名称太长
}

// 改成下面这样
use a::series::of::nested_modules;

fn main() {
    nested_modules();
}

// 导入枚举
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow}; // 多个可以使用 {}

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}

// 使用 * 导入全部
use TrafficLight::*;

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```


### 使用 super 访问父模块

下面的代码中 tests 模块和 client 模块同级, 不能直接通过 `client::connect()` 来访问.

```rust
pub mod client;

pub mod network;

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        ::client::connect(); // 从根模块开始
        super::client::connect(); // super 表示父模块
    }
}
```

如果从 super 开始还是很长, 可以使用 use 来缩短:

```rust
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```

### 使用 pub use 重导出模块

假设模块中有两个子模块:

src/lib.rs:

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

使用时需要这样:

```rust
extern crate art;

use art::kinds::PrimaryColor; // 不方便
use art::utils::mix;          // 不方便

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

使用 `pub use` 重导出子模块和子模块中的函数:

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

使用时就可以这样:

```rust
extern crate art;

use art::PrimaryColor; // 不需要子模块的名称了
use art::mix;

fn main() 
    // --snip--
}
```



------------------------------
# 工作空间

在 `add/Cargo.toml` 中：

```toml
[workspace]

members = [
    "adder",
    "add-one"
]
```

在 add 目录下使用 `cargo new --bin adder` 创建一个名为 adder 的二进制项目。

在 add 下使用 `cargo new --lib add-one` 创建一个名为 add-one 的库项目。

现在 add 目录的文件结构如下：

```
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

在 `add-one/src/lib.rs` 中有一个 `add_one` 函数。

如果想在 adder 中使用 add-one, 需要在 `adder/Cargo.toml` 中加入下面的声明：

```toml
[dependencies]

add-one = { path = "../add-one" }
```

这样就可以在 adder 中像下面这样使用 add-one 了：

```rust
extern crate add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}!", num, add_one::add_one(num));
}
```

在 add 目录下
* 使用 `cargo build` 构建所有的项目。
* 使用 `cargo run -p adder` 来运行 adder 项目。

如果在 add-one 中引入外部 crate, 可以在 `add-one/Cargo.toml` 中加入下面的依赖：

```
[dependencies]

rand = "0.3.14"
```







-----------------------------

# 错误处理


## 触发panic

使用 `panic!` 宏来抛出异常:

```rust
panic!("crash and burn");
```

运行时 `RUST_BACKTRACE=1 cargo run` 来打印出详细的错误堆栈.


## Result 

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

示例: 打开文件

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

open() 返回的类型是 `std::result::Result<std::fs::File, std::io::Error>`.

Result 和 Option 一样都被导入到了 prelude 中, 所以可以直接使用 Ok, Err 枚举值.


## 匹配不同的错误

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

上例中当错误为 NotFound 时会尝试创建文件.

ref 是必须的，这样 error 就不会被移动到 guard 条件中而仅仅只是引用它.


## unwrap 和 expect

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

`unwrap` 是 Result 提供的一个方法, 如果 Result 匹配到了 Ok, 那么 unwrap 就返回 Ok 中的内容, 如果匹配到了 Err, 那么就 panic.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

`expect` 中我们可以自定义 panic 时的错误信息.


## 传播错误

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e), // 这里的 return 是结束了整个函数
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

上例是从文件中读取用户名, 当读取失败时返回错误而不是 panic.

下例是上例的一个简写形式.

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?; // 多了个 ?
    let mut s = String::new();
    f.read_to_string(&mut s)?; // 多了个 ?
    Ok(s)
}
```

`?` 的作用是: 如果 Result 返回了 Ok, 那么 `?` 就返回 Ok 中的内容, 如果返回了 Err, 就 return Err.


> ? 所使用的错误值被传递给了 from 函数，它定义于标准库的 From trait 中，其用来将错误从一种类型转换为另一种类型。到问号运算符调用 from 函数时，收到的错误类型被转换为定义为当前函数返回的错误类型。这在当一个函数返回一个错误类型来代表所有可能失败的方式时很有用，即使其可能会因很多种原因失败。只要每一个错误类型都实现了 from 函数来定义如将其转换为返回的错误类型，问号运算符会自动处理这些转换。


还可以链式调用:

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

`?` 只能用于返回 Result 的函数, 比如下面的 main 函数没有返回值, 使用 `?` 会导致程序 panic:

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?; // 这里不能使用 ?
}
```

------------------------------
# trait
- 类似接口, 一组方法的集合.


## 定义trait

```rust
pub trait Summarizable {
    fn summary(&self) -> String;
}
```

+ 方法声明的最后需要添加分号 `;`.


## 实现trait

```rust
fn main() {
    let news_article = NewsArticle {
        headline: String::from("headline"),
        author: String::from("chen"),
    };
    println!("{}", news_article.summary());

    let tweet = Tweet {
        username: String::from("chen"),
        content: String::from("content"),
    };
    println!("{}", tweet.summary());
}

pub trait Summarizable {
    fn summary(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub author: String,
}

impl Summarizable for NewsArticle {
    fn summary(&self) -> String {
        format!("{}, by {}", self.headline, self.author)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summarizable for Tweet {
    fn summary(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

默认实现:

```rust
pub trait Summarizable {
    fn summary(&self) -> String {
        String::from("(Read more...)")
    }
}

pub struct NewsArticle {
    pub headline: String,
    pub author: String,
}

impl Summarizable for NewsArticle {}

fn main() {
    let news_article = NewsArticle {
        headline: String::from("headline"),
        author: String::from("chen"),
    };
    println!("{}", news_article.summary());
}
```

调用trait中的其他方法:

```rust
pub trait Summarizable {
    fn author_summary(&self) -> String;

    fn summary(&self) -> String {
        format!("(Read more from {}...)", self.author_summary())
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summarizable for Tweet {
    fn author_summary(&self) -> String {
        format!("@{}", self.username)
    }
}

fn main() {
    let tweet = Tweet {
        username: String::from("chen"),
        content: String::from("content"),
    };
    println!("{}", tweet.summary());
}
```


------------------------------
# 泛型

## 定义泛型

**结构体中的泛型**:

```rust
// 一种类型
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}

// 多种类型
struct Point<T, U> {
    x: T,
    y: U,
}
```


**枚举中的泛型**:

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```


**函数中的泛型**:

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

`impl<T>` 中的 T 是必须的, 这是告诉编译器 Point 后的 T 是泛型而不是具体类型.

下面展示了为具体类型实现的方法:

```rust
impl Point<f32> { // 为 T = f32 实现的方法
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

下例中的 mixup 使用两个不同类型的 Point 组装成一个新的 Point:

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> { // impl 后的 T, U 是相对于结构体的
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> { // mixup 后的 V, W 是相对于方法的
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

## Trait Bounds

```rust
pub trait Summarizable {
    fn author_summary(&self) -> String;

    fn summary(&self) -> String {
        format!("(Read more from {}...)", self.author_summary())
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summarizable for Tweet {
    fn author_summary(&self) -> String {
        format!("@{}", self.username)
    }
}

pub fn notify<T: Summarizable>(item: T) {
    println!("Breaking news! {}", item.summary())
}

fn main() {
    let tweet = Tweet {
        username: String::from("chen"),
        content: String::from("content"),
    };
    notify(tweet)
}
```

可以通过 `+` 来为泛型指定多个 trait bounds: `T: Summarizable + Display`, 需要两个 trait 都实现.

可以为每个泛型都指定 trait: `fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {`, 为了方便阅读, 可以将泛型声明放到 where 从句中:

```rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

## 不同泛型实现不同的方法

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

`Pair<T>` 总是实现了 new 方法, 但是只有当 T 实现了 Display 和 PartialOrd 时才有 `cmp_display` 方法.



------------------------------
# 测试

## 判断函数

```Rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert!(2 + 2 == 4); 
        assert_eq!(2 + 2, 4);
        assert_ne!(2 + 2, 3);
    }
}
```

`assert_eq!` 和 `assert_ne!` 在底层使用了 `==` 和 `!=`, 并且在断言失败时会使用调试格式打印出参数, 所以被比较的值需要实现 `PartialEq` 和 `Debug` trait.


### should_panic

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

确保错误信息中包含指定的字符串:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be between 1 and 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```




## 自定义错误信息

```rust
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

除了所需的参数外, 其余参数会被传递给 `format!` 宏.


## 引用外部函数

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

因为测试代码在 tests 模块中, 为了直接使用外部模块的函数, 使用了 `use super::*;`.


## 运行测试

`cargo test` 并行地运行所有测试.

`--` 是分隔符, 之前的参数会传递给 `cargo test`, 之后的参数会传递给测试二进制文件. 运行 `cargo test --help` 会告诉你 cargo test 的相关参数，而运行 `cargo test -- --help` 则会告诉你位于分隔符 -- 之后的相关参数.

多个测试默认使用线程来并行的运行, 如果不希望并行, 可以使用 `cargo test -- --test-threads=1` 来串行执行.

如果测试通过了, 测试库默认会捕获打印到标准输出的任何内容, 不会显示出来. 如果测试失败了则会显示出来(显示失败的那个函数的输出, 成功的不会显示).

运行特定的测试: `cargo test func_name`(执行包含 `func_name` 的测试).

忽略某些测试:

```rust
#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

如果想执行这些被忽略的测试, 可以使用 `cargo test -- --ignored`.


## 测试的组织结构

### 单元测试

`#[cfg(test)]` 模块只在运行 `cargo test` 时才会被编译, 运行 `cargo build` 时不会被编译, 节省空间.

### 集成测试

创建一个 tests 目录, 与 src 目录同级.

tests/integration_test.rs:

```rust
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

每个文件都会被编译为单独的 crate.

不需要 `#[cfg(test)]`, 编译器只会在运行 `cargo test` 时编译这个目录中的文件.

运行特定集成测试文件中的所有测试: `cargo test --test <file_name>`.

将辅助代码移动到单独的模块中, 比如 `tests/common/mod.rs`(common 作为一个模块, 不会出现在测试结果中), 然后就可以在测试代码中调用这些辅助函数了:

tests/integration_test.rs:

```rust
extern crate adder;

mod common; // 模块声明

#[test]
fn it_adds_two() {
    common::setup(); // 调用
    assert_eq!(4, adder::add_two(2));
}
```


# 并发 concurrency

并发编程（Concurrent programming），代表程序的不同部分相互独立的执行，而 并行编程（parallel programming）代表程序不同部分于同时执行.

编程语言有一些不同的方法来实现线程。很多操作系统提供了创建新线程的 API。这种由编程语言调用操作系统 API 创建线程的模模型有时被称为 1:1，一个 OS 线程对应一个语言线程。

很多编程语言提供了自己特殊的线程实现。编程语言提供的线程被称为 绿色（green）线程，使用绿色线程的语言会在不同数量的 OS 线程中执行它们。为此，绿色线程模式被称为 M:N 模型：M 个绿色线程对应 N 个 OS 线程，这里 M 和 N 不必相同。


## 使用 spawn 创建新线程

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

主线程结束时子线程也结束了, 不管其有没有执行完.


## 使用 join 等待线程结束

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

spawn 返回的类型是 JoinHandle, JoinHandle 是一个拥有所有权的值. join() 会等待线程执行完毕才返回.


## 线程与 move 闭包

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

上面的例子会报错. Rust 会推断如何捕获 v，因为 println! 只需要 v 的引用，闭包尝试借用 v。然而这有一个问题：Rust 不知道这个新建线程会执行多久，所以无法知晓 v 的引用是否一直有效。

正确的做法是在闭包前加 move, 这样 v 的所有权就被转移到闭包内了:

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```


## 使用 channel 在线程间传递数据

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

使用 `mpsc::channel` 函数创建一个新的通道；mpsc 是 **多个生产者，单个消费者（multiple producer, single consumer）** 的缩写。

mpsc::channel 函数返回一个元组：第一个元素是发送端，而第二个元素是接收端。由于历史原因，tx 和 rx 通常作为 发送者（transmitter）和 接收者（receiver）的缩写.

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap(); // 如果接收端被丢弃了将返回结果, 这里会 panic.
        // println!("val is {}", val); // 会报错, val 的所有权已经被转移了.
    });

    let received = rx.recv().unwrap();
    println!("Got {}", received);
}
```

`rx.recv()` 会阻塞线程直到有值或者出错. `rx.try_recv()` 则不会阻塞.

`tx.send(val)` 会获取 val 的所有权, 所以在这之后就不能再次使用 val 了.

发送多个值:

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got {}", received);
    }
}
```


多个生产者:

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    let tx1 = mpsc::Sender::clone(&tx); // 复制通道
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got {}", received);
    }
}
```

## 互斥器 mutex

互斥器（mutex）是 “mutual exclusion” 的缩写，也就是说，任意时刻，其只允许一个线程访问某些数据。

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);
    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }
    println!("m = {:?}", m);
}
```

lock 会阻塞当前线程, 直到我们获取了锁. 如果另一个线程拥有锁，并且那个线程 panic 了，则 lock 调用会失败。在这种情况下，没人能够再获取锁，所以这里选择 unwrap 并在遇到这种情况时使线程 panic。

一旦获取了锁，就可以将返回值（在这里是num）视为一个其内部数据的可变引用了。

`Mutex<T>` 是一个智能指针。更准确的说，lock 调用返回一个叫做 `MutexGuard` 的智能指针。这个智能指针实现了 `Deref` 来指向其内部数据；其也提供了一个 `Drop` 实现当 MutexGuard 离开作用域时自动释放锁。为此，我们不会冒忘记释放锁并阻塞互斥器为其它线程所用的风险，因为锁的释放是自动发生的。


### 多个线程间使用 mutex 访问数据

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap();
    }
    println!("result is {}", *counter.lock().unwrap());
}
```

Mutex 需要在多个线程中保留所有权, 因此这里使用 Arc 来允许有多个所有者. Arc 是 atomic Rc, 可以在多个线程中使用.



