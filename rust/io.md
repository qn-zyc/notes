<!-- TOC -->

- [io](#io)
    - [参数](#参数)
        - [读取命令行参数值](#读取命令行参数值)
        - [环境变量](#环境变量)
    - [标准输出](#标准输出)
    - [文件](#文件)
        - [读取](#读取)

<!-- /TOC -->


# io

------------------------------
## 参数

### 读取命令行参数值

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect(); // 获取参数并转换为集合
    println!("{:?}", args);
}
```

执行:

```bash
> cargo run
["target/debug/minigrep"]

> cargo run abc 123
["target/debug/minigrep", "abc", "123"]
```


### 环境变量

```rust
use std::env;

fn main() {
    let case_sensitive = env::var("CASE_SENSITIVE");
}
```


------------------------------
## 标准输出

```rust
println!("hello") // 打印到标准输出中
eprintln!("error") // 打印到标准错误输出中
```


------------------------------
## 文件

### 读取

```rust
use std::fs::File;
use std::io::prelude::*;

fn main() {
    // 打开文件, 读取文件内容
    let mut f = File::open(filename).expect("file not found");
    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");
    println!("With text:\n{}", contents);
}
```

`std::io::prelude::*` 包含许多对应 io 有帮助的 trait.