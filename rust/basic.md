<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [install](#install)
	- [hello world](#hello-world)
	- [使用 Cargo](#使用-cargo)
- [表达式和语句](#表达式和语句)
- [注释](#注释)
- [所有权](#所有权)
	- [Copy 类型](#copy-类型)
- [引用和借用](#引用和借用)
	- [&mut 引用](#mut-引用)
	- [借用的规则](#借用的规则)
- [生命周期](#生命周期)
	- [static生命周期](#static生命周期)
	- [隐式生命周期](#隐式生命周期)
	- [显式生命周期](#显式生命周期)
		- [声明多个生命周期](#声明多个生命周期)
	- [结构体中的生命周期](#结构体中的生命周期)
- [参考](#参考)

<!-- /TOC -->

# install

1. 安装: `curl https://sh.rustup.rs -sSf | sh`
2. 设置 PATH: `source $HOME/.cargo/env`
3. 显示版本: `rustc --version`
4. 卸载: `rustup self uninstall`

更新 rust: `rustup update`

## hello world

main.rc:

```rust
fn main() {
    println!("Hello, world!");
}
```

编译运行:

```bash
$ rustc main.rs
$ ./main
Hello, world!
```


## 使用 Cargo

* 查看版本: `cargo --version`
* 创建新项目: `cargo new hello_world --bin`(`--bin`: 创建可执行项目而不是库)
* build(不执行): `cargo build`(在`hello_world`目录下)
* build+执行: `cargo run`
* 发布构建: `cargo build --release`(会优化代码)


# 表达式和语句

表达式返回一个值, 而语句不是. `x + 1;` 语句不返回值, `x + 1` 表达式则返回一个值.

Rust 主要是基于表达式的语句, 只有两种语句, 其他都是表达式. 两种语句是: 声明语句和表达式语句.

`let x = (let y = 1);` 这种会导致编译失败, 因为 `let y = 1` 是语句而不是表达式, 它没有返回值.

赋值一个绑定过的变量(`y = 2`) 是一个表达式, 它返回空的元组 `()`, 所以 `let y = (x = 1)` 后 y 的值是 `()`.

函数调用, 宏调用都是表达式.

`{}` 也是表达式:

```rust
let y = { // 花括号是一个表达式, 返回值 4.
	let x = 3;
	x + 1 // 表达式不带分号.
}
```


# 注释

Rust 有两种注释, 行注释(line comments) 和 文档注释(doc comments).

行注释以 `//` 开头, 文档注释以 `///` 开头, 文档注释支持 Markdown 语法.

还有一种文档注释以 `//!` 开头, 常用来注释模块.



# 所有权

所有权规则:
- Rust 中每一个值都有一个称之为其 所有者（owner）的变量。
- 值有且只能有一个所有者。
- 当所有者（变量）离开作用域，这个值将被丢弃。


```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

上面的例子编译时会报错, 原因是 s1 被 move 到了 s2, 所以 s1 不能再被使用了。 在栈上保存着字符串的基本信息（指针， 长度， 容量等）， 在堆上存放这字符串数据, 当执行 `let s2 = s1;` 时会将 s1 的栈上的内容拷贝一份给 s2, 所以 s1 和 s2 都引用了堆上相同的内存. 当离开作用域时, Rust 会尝试回收 s1 和 s2 所占用的内存, 这时会出现同一内存被释放两次. 为了避免这个问题, Rust 会将 s1 标记为无效的, 也就是说堆上的内存的所有权移动到 s2 了, s1 成为无效的了.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

上例调用了 clone(), 这样 s2 会拥有一份 s1 堆内容的拷贝.


## Copy 类型

- 如果一个类型拥有 Copy trait，一个旧的变量在将其赋值给其他变量后仍然可用。
- Rust 不允许自身或其任何部分实现了 Drop trait 的类型使用 Copy trait。
- 作为一个通用的规则，任何简单标量值的组合可以是 Copy 的，任何需要分配内存，或者本身就是某种形式资源的类型不会是 Copy 的。

如下是以下 Copy 的类型:
- 所有整数类型，比如 u32。
- 布尔类型，bool，它的值是 true 和 false。
- 所有浮点数类型，比如 f64。
- 元组，当且仅当其包含的类型也都是 Copy 的时候。(i32, i32) 是 Copy 的，不过 (i32, String) 就不是。

```rust
let v1 = 1;
let v2 = v1;
println!("{}", v1);
```

上例中, 我们在将 v1 转移到 v2 后, 还是可以使用 v1, 这是因为所有的基本类型都实现了 `Copy` 特性, 它的拷贝是完整拷贝.


# 引用和借用
- 引用就像是创建了一个指向栈上元数据的指针, 不会获取所有权.
- 我们将获取引用作为函数参数称为 借用（borrowing）。
- 借用主要是为了解决这样的问题: 所有权发生改变之后原来的绑定就不能使用资源了.
- `&`: 不可变引用.
- `&mut`: 可变引用.
- 借用避免的是数据竞争问题.
- 借用的类型, 比如借用 i32, 那么借用的类型就是 `&i32`. 再比如 `&str`.

借用默认是不能修改借用的值的:

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world"); // !Error
}
```

## 可变引用

可变引用, 可以改变引用的资源.

```rust
let mut x = 1;
{
    let y = &mut x;
    *y += 1;
}
println!("x = {}", x); // x = 2
```

x 也必须被标记为可变的, 否则无法获取它的引用. y 要获取引用的值需要加 `*`.

一个在函数中使用可变引用的示例：

```rust
fn incr(v: &mut i32) {
    *v += 1;
}

fn main() {
    let mut x = 1i32;
    incr(&mut x);
    println!("x = {}", x);
}
```

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world"); // 可以修改借用的值
}
```

在特定作用域中的特定数据有且只有一个可变引用:

```rust
let mut s = String::from("hello");

let r1 = &mut s; 
let r2 = &mut s; // !Error
```

不能在拥有不可变引用的同时拥有可变引用:

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // !BIG PROBLEM
```


## 引用的规则
1. 在任意给定时间，只能 拥有如下中的一个：
- 一个可变引用。
- 任意数量的不可变引用。
2. 引用必须总是有效的。

违反第二条的示例如下:

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```


下面的例子说明了第一个规则:

```rust
let y: &i32;
let x = 5;
y = &x;
```

x 没有 y 活得久, 所以 y 不能借用 x.


下面是一个错误的例子, 说明了第二条规则:

```rust
let mut x = 1;
let y = &mut x;
*y += 1;
println!("x = {}", x); // cannot borrow `x` as immutable because it is also borrowed as mutable
```

在这个作用域内, 我们拥有了一个指向 x 的可变引用 y, 所有打印那行不允许再创建不可变引用 `&T` 了.


# 生命周期
- 声明周期主要解决的问题是: 怎么将小作用域内的变量的作用域扩展到大作用域中.
- `'a` 读作生命周期a。
- 生命周期只有在引用类型中才有用。


## static生命周期

`'static` 生命周期横跨整个程序，比如下面这样的：

```rust
let s: &'static str = "hello";
```

基本字符串是 `'static str` 类型的， 因为它的引用一直有效。它被写入了最终的库文件的数据段。

另一个例子是全局量：

```rust
static FOO: i32 = 1234;
let x: &'static i32 = &FOO;
```


## 隐式生命周期

```rust
fn main() {
    let x = 1;
    let y = foo(&x);
    println!("{}", y);
}

fn foo(x: &i32) -> &i32 {
    x
}
```

foo 函数的参数和返回值是一个引用，它包含了隐式的生命周期命名，由编译器自动推导：

```rust
fn foo<'a>(x: &'a i32) -> &'a i32 {
    x
}
```

foo 后面的 `<'a>` 属于泛型声明，生命周期也是一种泛型。上面的例子约束：**返回值的 lifetime 要小于或等于参数 x 的 lifetime**。比如下面这样的：

```rust
fn foo<'a>(x: &'a i32) -> &'a i32 {
    &123
}
```

因为常量的 lifetime 是 `'static`, 但返回值不依赖于输入值，所以返回值的生命周期不管怎样都满足。

规则：
- 如果参数省略了生命周期，那么每个参数的生命周期都将是不同的。




## 显式生命周期

```rust
fn foo(x: &i32, y: &i32) -> &i32 {
    if true {
        x
    } else {
        y
    }
}
```

上例会报 `missing lifetime specifier` 错误。对应编译器来说，它要确保返回值的生命周期比参数x和y都要短，但是现在它没法确认。

这时就需要我们显示指定他们的生命周期之间的关系：

```rust
fn main() {
    let x = 1;
    let y = 2;
    println!("{}", foo(&x, &y));
}

fn foo<'a>(x: &'a i32, y: &'a i32) -> &'a i32 {
    if true {
        x
    } else {
        y
    }
}
```

我们声明了一个生命周期 `'a`，并且输出值的生命周期和输入值一样，所以满足上面的条件。

**当输出值R依赖输入值X，Y，Z时，当且仅当输出值的生命周期为所有输入值的生命周期的交集的子集时，生命周期合法。**

也就是说，返回值的生命结束之前，所依赖的参数不能先挂了，不然的话指针就指向了被回收的内存（悬垂指针）。


### 声明多个生命周期

```rust
fn foo<'a, 'b>(x: &'a i32, y: &'b i32) -> &'a i32 {
    if true {
        x
    } else {
        y
    }
}
```

上面的例子编译出错：`lifetime mismatch`。编译器没法确认a和b的生命周期的交集，所以返回y时就不确定y是否匹配a（需要确保b活得比a长才行，这样a和b的交集就是a了）。

因此声明b时需要告诉编译器a是b的子集（b活得比a长或相等）：`'b: 'a`:

```rust
fn main() {
    let x = 1;
    let y = 2;
    println!("{}", foo(&x, &y));
}

fn foo<'a, 'b: 'a>(x: &'a i32, y: &'b i32) -> &'a i32 {
    if true {
        x
    } else {
        y
    }
}
```


## 结构体中的生命周期











# 参考
- [官网](https://www.rust-lang.org/zh-CN/index.html)
- [Rust程序设计语言](https://kaisery.gitbooks.io/rust-book-chinese/content/)
- [生命周期](http://wiki.jikexueyuan.com/project/rust-primer/ownership-system/lifetime.html)
