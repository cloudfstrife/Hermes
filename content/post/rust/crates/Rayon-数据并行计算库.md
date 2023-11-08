---
title: "Rayon - 数据并行计算库"
date: 2023-03-31T11:05:38+08:00
lastmod: 2023-03-31T11:05:38+08:00

categories:
  - Rust
tags:
  - Rust
  - Rayon
keywords: 
  - Rust
  - Rayon

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

[Rayon](https://github.com/rayon-rs/rayon) 是一个用于 Rust 实现的轻量级数据并行化库。使用 Rayon 可以很容易地将顺序计算转换为并行计算，并保证数据竞争的自由。

<!--more-->

## 并行迭代器

要使用并行迭代器，首先通过在模块中添加 `use rayon::prelude::*` 来导入特征。然后只需将 `.iter()` 及类似的调用改为 `.par_iter()` ， `.par_iter_mut()` 或 `.into_par_iter` 来获取并行迭代器。并行迭代器和常规迭代器一样，它的工作方式是先构造一个计算，然后执行。

```rust
use rayon::prelude::*;
fn sum_of_squares(input: &[i32]) -> i32 {
    input.par_iter() // <-- just change that!
         .map(|&i| i * i)
         .sum()
}
```

并行迭代器自行决定如何将数据划分为任务；它将动态地调整以实现性能的最优化。如果应用程序需要更好的灵活性，Rayon 还提供了 `join` 和 `scope` 函数，允许应用程序创建并行任务。

参见  
join : [https://docs.rs/rayon/1.7.0/rayon/fn.join.html](https://docs.rs/rayon/1.7.0/rayon/fn.join.html)  
scope : [https://docs.rs/rayon/1.7.0/rayon/fn.scope.html](https://docs.rs/rayon/1.7.0/rayon/fn.scope.html)


## 自定义线程池

Rayon 默认使用 Rayon 的全局线程池。为了获得更多的控制权，应用程序可以创建自定义线程池。

```rust
fn main() -> Result<(), rayon::ThreadPoolBuildError> {
    let pool = rayon::ThreadPoolBuilder::new()
        .num_threads(256)
        .build()
        .unwrap();
    pool.install(|| println!("Hello from my custom thread!"));
    Ok(())
}
```