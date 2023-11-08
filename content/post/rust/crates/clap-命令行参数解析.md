---
title: "clap - 命令行参数解析"
date: 2023-04-01T08:42:23+08:00
lastmod: 2023-04-01T08:42:23+08:00

categories:
  - Rust
tags:
  - Rust
  - JSON
keywords: 
  - Rust
  - JSON

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

[clap](https://crates.io/crates/clap) 是一个易于使用、高效且功能齐全的用于在解析子命令和命令行参数的 crate 。 clap会自动处理其余的繁杂工作。 这样工程师可以把时间和精力放在实现程序功能上，而不是参数的解析和验证上。

<!--more-->

## 注解参数解析

```console
cargo add clap --features derive
```

```rust
use clap::{Parser, Subcommand, ValueEnum};

#[derive(Parser)]
// #[command(author,version,about,long_about=None)]
#[command(name = "testing")]
#[command(author)]
#[command(version = "1.0")]
#[command(about = "clap annotation test", long_about = None)]
struct Testing {
    #[arg(value_enum,long, short,default_value_t=LogLevel::INFO)]
    log_level: LogLevel,

    #[command(subcommand)]
    sub: Option<Action>,
}

#[derive(Debug, Clone, Copy, ValueEnum)]
enum LogLevel {
    DEBUG,
    TRACE,
    INFO,
    WARNING,
    ERROR,
    PANIC,
}



#[derive(Subcommand)]
enum Action {
    Scan {
        #[arg(long, short = 's',default_value_t=String::from("127.0.0.1"))]
        host: String,
        #[arg(long, short)]
        port: Option<u32>,
    },
    Attack {},
}

fn main() {
    let cmd = Testing::parse();
    println!("{:?}", cmd.log_level);
    match cmd.sub {
        Some(action) => match action {
            Action::Scan { host, port } => {
                println!("scanning {} {}", host, port.unwrap_or_default())
            }
            Action::Attack {} => {
                println!("attacking")
            }
        },
        None => {
            println!("run testing with out subcommand")
        }
    }
}
```

主要的步骤：

1. 定义接收参数的类型，如果有子命令，定义子命令 enum
2. 为定义的接收参数的类型增加 `#[derive(Parser)]`
3. 跟据需要添加`#[command(version = "1.0")]`注解，如果值为空，则会从 `Cargo.toml` 中解析
3. 如果有必要，为类型的属性添加 `#[arg(long, short = 's',default_value_t=String::from("127.0.0.1"))]` 注解
4. 在 main 函数中调用类型的 `parse()` 函数
5. 跟据解析出的类型，执行对应的操作
