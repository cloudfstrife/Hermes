---
title: "使用Criterion对Rust代码进行基准测试"
date: 2024-06-25T15:38:47+08:00
lastmod: 2024-06-25T15:38:47+08:00

categories:
  - rust
tags:
  - rust
  - Benchmark
  - Criterion
keywords: 
  - rust
  - Benchmark
  - Criterion

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

基准测试是用于衡量软件性能的测试，是软件测试方面的一个关键技术。Rust 在 Nightly 版本中提供了基本的基准测试（截止本文书写时间），但是并没有进入 Stable 。

<!--more-->

## 简介

[Criterion](https://crates.io/crates/criterion) 是一个强大的 Rust 基准测试 Crates ，提供了对代码性能精确可靠的分析和其它各种各样的特性。从基本的基准测试，到高级的性能统计与分析，Criterion 都可以完成。

## 在 Rust 项目中使用 Criterion

### 创建项目

```bash
cargo new --lib testing 
```

### 实现逻辑代码

```rust 
// src/lib.rs 
use std::cmp::Ordering;

pub fn binary_search(v: &[i64], target: i64, begin: usize, end: usize) -> Result<usize, &str> {
    if end < begin {
        return Err("NO FOUND");
    }

    let mid = begin + (end - begin) / 2;

    match v[mid].cmp(&target) {
        Ordering::Equal => Ok(mid),
        Ordering::Less => return binary_search(v, target, mid + 1, end),
        Ordering::Greater => return binary_search(v, target, begin, mid - 1),
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_binary_search() {
        let v = &[1, 2, 3, 4, 5, 6];

        let success = binary_search(v, 2, 0, v.len() - 1);
        assert!(success.is_ok());
        assert_eq!(1, success.unwrap());

        let failed = binary_search(v, 10, 0, v.len() - 1);
        assert!(failed.is_err());
    }
}
```

### 单元测试 

```bash
cargo test 
```

### 增加 criterion 

```bash
cargo add --dev criterion -F html_reports
```

### 创建基准测试代码文件

```bash
mkdir -p benches
touch benches/binary_search_benchmark.rs
```

### 编写基准测试代码

```rust
// benches/binary_search_benchmark.rs
use criterion::{criterion_group, criterion_main, Criterion};

fn binary_search_benchmark(c: &mut Criterion) {
    let v = &[1, 2, 3, 4, 5, 6];
    let end = v.len() - 1;

    c.bench_function("binary_search", |b| {
        b.iter(|| testing::binary_search(v, 2, 0, end))
    });
}

criterion_group!(benches, binary_search_benchmark);
criterion_main!(benches);
```
说明：

> `criterion_group!(benches, binary_search_benchmark);`：设置一个名为`benches`的基准组，其中包含 binary_search_benchmark 函数
>
> `criterion_main!(benches);` ： 生成一个运行基准测试的 `main` 函数


### 在 Cargo.toml 中注册 benchmark 测试

```toml
[[bench]]
name = "binary_search_benchmark"
harness = false
```

### 安装GNUPlot

> GNUPlot 是生成图表的工具，也可以不安装，如果没找到 GNUPlot，会使用 [plotters](https://crates.io/crates/plotters) 来生成

```bash
sudo apt install gnuplot
```

### 执行基准测试

```bash
cargo bench
```

此时在 `target/criterion/report/index.html` 就是生成的基准测试报告。

## 报告解读 

在浏览器中打开报告，可以看到两张图。

左图显示了函数调用的时间分布和可信区间。左图下方是附加统计信息：


* Slope - 线性回归斜率的置信区间
* R^2 - 置信区间下限和上限的拟合优度值
* Mean - 平均值
* Std. Dev. - 每次迭代时间的标准偏差，如果此值偏大，说明本次基准测试存在噪声
* Median - 中位数
* MAD - 中位绝对偏差，如果此值偏大，说明本次基准测试存在噪声


右图显示了随着迭代次数增多（即代码中b的数量增大），基准测试所花费时间的变化。

## 基准测试的应用

当对一个函数进行优化之后，再次运行 `cargo bench` ， 生成的报告下方会出现 `Change Since Previous Benchmark` 的节，这一节表示两次基准测试的变化，图中红色部分表示上次的统计结果，蓝包部分表示最新一次的统计结果。对比两次的统计数据，即可以分析优化的影响。