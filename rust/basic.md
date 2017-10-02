<!-- TOC -->

- [install and uninstall rust](#install-and-uninstall-rust)
    - [hello world](#hello-world)
- [使用 Cargo](#使用-cargo)
- [变量](#变量)
    - [类型](#类型)
- [参考](#参考)

<!-- /TOC -->

# install and uninstall rust
1. 安装: `curl https://sh.rustup.rs -sSf | sh`
2. 设置PATH: `source $HOME/.cargo/env`
3. 显示版本: `rustc --version`
4. 卸载: `rustup self uninstall`

## hello world
main.rc:
```rust
fn main() {
    println!("Hello, world!");
}
```

编译运行:

```shell
$ rustc main.rs
$ ./main
Hello, world!
```


# 使用 Cargo
* 查看版本: `cargo --version`
* 创建新项目: `cargo new hello_world --bin`(`--bin`: 创建可执行项目而不是库)
* build(不执行): `cargo build`(在`hello_world`目录下)
* build+执行: `cargo run`
* 发布构建: `cargo build --release`(会优化代码)


# 变量

```rust
let x = 5;
let (x, y) = (1, 2); // 模式
let x: i32 = 5; // 指定类型
```

* 支持模式.
* `let x = 5;` 执行后x是不可变的(不能`x=6;`), 想要可变: `let mut x = 5; x = 6;`.
* 变量未初始化前不能使用: `let x: i32; println!("{}", x);` 是错误的!
* 变量的作用域只在声明它的块中, 子块中变量可以覆盖父块中同名变量.


## 类型
* 有符号整型: `i8, i16, i32, i64`
* 无符号整型: `u8, u16, u32, u64`





# 参考
* [官网](https://www.rust-lang.org/zh-CN/index.html)
* [Rust程序设计语言](https://kaisery.gitbooks.io/rust-book-chinese/content/)