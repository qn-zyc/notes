<!-- TOC -->

- [install](#install)
    - [hello world](#hello-world)
    - [使用 Cargo](#使用-cargo)
- [表达式和语句](#表达式和语句)
- [注释](#注释)
- [所有权](#所有权)
    - [Copy 类型](#copy-类型)
- [引用和借用](#引用和借用)
    - [可变引用](#可变引用)
    - [引用的规则](#引用的规则)
- [生命周期](#生命周期)
    - [规则](#规则)
    - [static生命周期](#static生命周期)
    - [隐式生命周期](#隐式生命周期)
    - [显式生命周期](#显式生命周期)
        - [声明多个生命周期](#声明多个生命周期)
    - [结构体中的生命周期](#结构体中的生命周期)
- [智能指针](#智能指针)
    - [Box<T>](#boxt)
        - [使用 Box 创建递归类型](#使用-box-创建递归类型)
    - [Deref trait](#deref-trait)
        - [隐式解引用强制多态](#隐式解引用强制多态)
    - [Drop trait](#drop-trait)
    - [Rc<T> 引用计数智能指针](#rct-引用计数智能指针)
    - [RefCell<T>](#refcellt)
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

* [Cargo 文档](https://doc.rust-lang.org/cargo/)
* 查看版本: `cargo --version`
* 创建新项目: `cargo new hello_world --bin`(`--bin`: 创建可执行项目而不是库), `cargo new --lib communicators`(创建库)
* build(不执行): `cargo build`(在`hello_world`目录下)
* build+执行: `cargo run`
* 发布构建: `cargo build --release`(会优化代码)
* 从 crates.io 安装二进制包到本地: `cargo install ripgrep`(安装名为 ripgrep 的二进制包)

Cargo 被设计为可以通过新的子命令而无须修改 Cargo 自身来进行扩展。如果 $PATH 中有类似 cargo-something 的二进制文件，就可以通过 `cargo something` 来像 Cargo 子命令一样运行它。像这样的自定义命令也可以运行 `cargo --list` 来展示出来。




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


------------------------------
# 生命周期
- 声明周期主要解决的问题是: 怎么将小作用域内的变量的作用域扩展到大作用域中.
- `'a` 读作生命周期a。
- 生命周期只有在引用类型中才有用, 生命周期在 `&` 符号之后.
- 生命周期也是泛型的一种.

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```


## 规则

函数或方法的参数的生命周期被称为 输入生命周期（input lifetimes），而返回值的生命周期被称为 输出生命周期（output lifetimes）.

1. 每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
2. 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。
3. 如果方法有多个输入生命周期参数，不过其中之一因为方法的缘故为 `&self` 或 `&mut self`，那么 self 的生命周期被赋给所有输出生命周期参数。这使得方法编写起来更简洁。





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

`'a` 的声明周期等同于 x 和 y 中较小的那一个.

**当输出值R依赖输入值X，Y，Z时，当且仅当输出值的生命周期为所有输入值的生命周期的交集的子集时，生命周期合法。**

也就是说，返回值的生命结束之前，所依赖的参数不能先挂了，不然的话指针就指向了被回收的内存（悬垂指针）。

当返回值不依赖于某个参数时, 可以不为那个参数指定生命周期:

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

当返回函数内部变量时, 最好返回一个有所有权的数据类型而不是一个引用:

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str() // Error
}
```


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

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```



------------------------------
# 智能指针

智能指针通常使用结构体实现。智能指针区别于常规结构体的显著特性在于其实现了 `Deref` 和 `Drop` trait。`Deref` trait 允许智能指针结构体实例表现的像引用一样，这样就可以编写既用于引用又用于智能指针的代码。`Drop` trait 允许我们自定义当智能指针离开作用域时运行的代码。


## Box<T>

Box 运行将值存放在堆上而不是栈上，留在栈上的是指向对数据的指针。

应用场景：

* 当有一个在编译时未知大小的类型，而又想要在需要确切大小的上下文中使用这个类型值的时候
* 当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候
* 当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型的时候

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b); // b = 5
}
```

当离开作用域的时候， b 将被释放， 包括栈上和堆上的数据。


### 使用 Box 创建递归类型

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

当使用上面的定义来定义一个递归类型时， Rust 会报错， 因为无法计算 List 需要占用多大空间。

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

这里 `Box<List>` 值占用一个指针大小的空间(usize)。


## Deref trait

重载解引用运算符 `*`。

Box 可以像引用那样使用 `*x` 来使用被引用的值:

```rust
let x = 5;
let y = Box::new(x);
println!("{}", *y); // 5
```

实现一个自己的 Box 类型:

```rust
use std::ops::Deref;

fn main() {
    let x = 6;
    let y = MyBox::new(x);
    println!("{}", *y);
}

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0 // 返回具体值的引用
    }
}
```

使用 `*y` 时 Rust 运行的是这样的代码: `*(y.deref())`。

### 隐式解引用强制多态

解引用强制多态（deref coercions）是 Rust 出于方便的考虑作用于函数或方法的参数的。其将实现了 Deref 的类型的引用转换为 Deref 所能够将原始类型转换的类型的引用。解引用强制多态发生于当作为参数传递给函数或方法的特定类型的引用不同于函数或方法签名中定义参数类型的时候，这时会有一系列的 deref 方法调用会将提供的类型转换为参数所需的类型。

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

这里传递 MyBox 的引用也能匹配 `&str` 类型是因为 Rust 实际做的是:

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

`*m` 获取了 String 类型， `&String[..]` 获取到的是 `&str`。

类似于如何使用 Deref trait 重载不可变引用的 `*` 运算符，Rust 提供了 DerefMut trait 用于重载可变引用的 `*` 运算符。

Rust 在发现类型和 trait 实现满足三种情况时会进行解引用强制多态：
* 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
* 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
* 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。

最后一个情况有些微妙：Rust 也会将可变引用强转为不可变引用。但是反之是 不可能 的：不可变引用永远也不能强转为可变引用。因为根据借用规则，如果有一个可变引用，其必须是这些数据的唯一引用（否则程序将无法编译）。将一个可变引用转换为不可变引用永远也不会打破借用规则。将不可变引用转换为可变引用则需要数据只能有一个不可变引用，而借用规则无法保证这一点。因此，Rust 无法假设将不可变引用转换为可变引用是可能的。



## Drop trait

* 不能显示调用 `drop()`。

一个示例:

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
}
```

运行结果:

```
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```

使用 `std::mem::drop` 调用对象的 drop():

```rust
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```


## Rc<T> 引用计数智能指针

* 引用计数（reference counting）的缩写
* 多所有权
* `Rc<T>` 用于当我们希望在堆上分配一些内存供程序的多个部分读取，而且无法在编译时确定程序的那一部分会最后结束使用它的时候。如果确实知道哪部分会结束使用的话，就可以令其成为数据的所有者同时正常的所有权规则就可以在编译时生效。
* `Rc<T>` 只能用于单线程场景

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a)); // clone 会增加引用计数
    let c = Cons(4, Rc::clone(&a));
}
```

使用 `Rc::strong_count()` 获取引用计数值:

```rust
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a)); // 1
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a)); // 2
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a)); // 3
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a)); // 2
}
```

`Rc<T>` 允许通过不可变引用来只读的在程序的多个部分共享数据。如果 `Rc<T>` 也允许多个可变引用，则会违反第四章讨论的借用规则之一：相同位置的多个可变借用可能造成数据竞争和不一致。


## RefCell<T>

* 对于引用和 `Box<T>`，借用规则的不可变性作用于编译时。对于 `RefCell<T>`，这些不可变性作用于运行时。对于引用，如果违反这些规则，会得到一个编译错误。而对于 `RefCell<T>`，违反这些规则会 panic!。
* `RefCell<T>` 正是用于当你确信代码遵守借用规则，而编译器不能理解和确定的时候。
* 类似于 `Rc<T>`，`RefCell<T>` 只能用于单线程场景

如下为选择 Box<T>，Rc<T> 或 RefCell<T> 的理由：

* Rc<T> 允许相同数据有多个所有者；Box<T> 和 RefCell<T> 有单一所有者。
* Box<T> 允许在编译时执行不可变（或可变）借用检查；Rc<T>仅允许在编译时执行不可变借用检查；RefCell<T> 允许在运行时执行不可变（或可变）借用检查。
* 因为 RefCell<T> 允许在运行时执行可变借用检查，所以我们可以在 **即便 RefCell<T> 自身是不可变的情况下修改其内部的值**(内部可变性模式)。








# 参考
- [官网](https://www.rust-lang.org/zh-CN/index.html)
- [Rust程序设计语言](https://kaisery.gitbooks.io/rust-book-chinese/content/)
- [生命周期](http://wiki.jikexueyuan.com/project/rust-primer/ownership-system/lifetime.html)
