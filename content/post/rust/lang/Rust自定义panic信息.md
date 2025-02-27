---
title: "Rust自定义panic信息"
date: 2025-02-26T18:13:41+08:00
lastmod: 2025-02-26T18:13:41+08:00

categories:
- Rust
tags:
- Rust
- Combinators
keywords:
- rust
- Combinators

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

在Rust里，默认情况下：`panic!`宏可以用来触发一个panic，它会立即终止当前执行的函数，并开始展开`[unwinding]`当前线程的调用栈，清理每个栈帧中的数据，清理完成后，它会终止当前运行的进程。

<!--more-->

## Panic的处理过程

Rust的错误处理流程如下图所示：

![rust-panic-process](/images/rust/rust-panic-process.svg)

其中 `panic_impl` 函数，由链接器在链接的时候，跟据不同的target链接到不同的处理。

## 自定义panic的处理

```rust
use std::panic;
use std::panic::PanicHookInfo;
use std::process;

fn main() {
    panic::set_hook(Box::new(panic_handler));
    panic!("version");
}

pub fn panic_handler(_panic_info: &PanicHookInfo) {
    println!("panic");
    process::exit(101);
}
```

以上代码的编译结果，只会输出 `panic` 这个词，不会输出其它的信息