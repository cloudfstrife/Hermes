---
title: "Linux环境Rust静态编译"
date: 2023-06-16T12:42:04+08:00
lastmod: 2023-06-16T12:42:04+08:00

categories:
  - Rust
tags:
  - Rust
keywords: 
 - rust

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

默认情况下，Rust 将静态链接所有 Rust 代码。但是，如果使用标准库，它将动态链接到系统的 libc 实现。

<!--more-->

如果想要100％静态二进制文件，可以在 Linux 上使用 MUSL libc。

## 安装 MUSL 支持

Rust 支持的平台 [Platform Support](https://doc.rust-lang.org/rustc/platform-support.html)


```console
rustup target add x86_64-unknown-linux-musl
```

## 使用 MUSL 构建

```console
cargo build --target x86_64-unknown-linux-musl
```