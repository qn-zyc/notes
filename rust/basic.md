
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [install and uninstall rust](#install-and-uninstall-rust)
	* [hello world](#hello-world)
* [使用 Cargo](#使用-cargo)
* [变量](#变量)
	* [类型](#类型)
* [参考](#参考)

<!-- /code_chunk_output -->


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


# 注释

Rust 有两种注释, 行注释(line comments) 和 文档注释(doc comments).

行注释以 `//` 开头, 文档注释以 `///` 开头, 文档注释支持 Markdown 语法.

还有一种文档注释以 `//!` 开头, 常用来注释模块.



# 所有权

Rust 中变量绑定有一个属性: 它们与它们所绑定的值的所有权. 这意味着一个绑定离开作用域后, 绑定的资源就会被释放.

Rust 确保了对于任何给定的资源都正好只有一个绑定与之对应.

```rust
let v1 = vec![1, 2, 3];
let v2 = v1;
println!("{}", v1[0])
```

上例中 v1 绑定的资源的所有权转移给 v2 后, 就不能再使用 v1 来访问资源了.

生成 vector 对象时, Rust 在堆上和栈上都会分配一些内存(我猜在栈上存储了一些基本信息, 比如长度什么的, 在堆上保存了具体的数据), 当执行 `let v2 = v1;` 时, 只是将栈上的内容拷贝了一份给 v2, 他们都执行堆上的数据. 如果通过 v2 改变了 vector 的大小, 那么 v1 在栈上的信息与堆上的信息就不一致了, 这可能导致段错误.


## Copy 类型

```rust
let v1 = 1;
let v2 = v1;
println!("{}", v1);
```

上例中, 我们在将 v1 转移到 v2 后, 还是可以使用 v1, 这是因为所有的基本类型都实现了 `Copy` 特性, 它的拷贝是完整拷贝.


# 引用和借用
- 借用主要是为了解决这样的问题: 所有权发生改变之后原来的绑定就不能使用资源了.
- `&`: 不可变引用.
- `&mut`: 可变引用.
- 借用避免的是数据竞争问题.
- 借用的类型, 比如借用 i32, 那么借用的类型就是 `&i32`. 再比如 `&str`.


```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = vec![3, 4, 5];
    let answer = foo(&v1, &v2);
    println!("answer are: {}", answer);
    // 这里还可以继续使用 v1, v2
}

fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    10
}
```

使用 `&Vec<i32>` 来获取引用, 传参时使用 `&v1` 来传递参数. `&T` 类型是一个引用, 它借用了所有权. **借用变量的绑定在离开作用域后并不释放资源**.

引用是不可变的, 所以在使用引用变量时不能改变值:

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    v1.push(4); // 错误!!!
    10
}
```


## &mut 引用

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


## 借用的规则

1. 任何借用必须位于比拥有者更小的作用域.
2. 对同一资源的借用, 以下情况不能同时出现在同一作用域下:
	- 1个或多个不可变引用(`&T`)
	- 唯一一个可变引用(`&mut T`)

第二条的意思是: 同一个作用域下, 要么只有一个可变引用(`&T`), 要么有N个不可变引用, 但是可变引用和不可变引用不能同时存在.

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
