---
title: "Rust中字符串与字节集合的转换"
date: 2023-06-15T16:11:13+08:00
lastmod: 2023-06-15T16:11:13+08:00

categories:
  - Rust
  - String
tags:
  - Rust
  - String
keywords: 
  - Rust
  - String

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Rust中字符串与字节集合的转换

<!--more-->

## String

String -> &str

```rust
let a:String = String::from("version");

let X:&str = &a;
let Y:&str = a.as_str();
```

String -> &[u8]

```rust
let a: String = String::from("version");

let X: &[u8] = a.as_bytes();
```

String -> Vec<u8>

```rust
let a: String = String::from("version");

let X: Vec<u8> = a.into_bytes();
```

## &str

&str -> String

```rust
let a: &str = "version";

let X: String = String::from(a);
let Y: String = a.to_string();
let Z: String = a.to_owned();
```

&str -> &[u8]

```rust
let a: &str = "version";

let X: &[u8] = a.as_bytes();
```

&str -> Vec<u8>

```rust
let a: &str = "version";

let X: Vec<u8> = a.as_bytes().to_vec();
let Y: Vec<u8> = a.as_bytes().to_owned();
```

## &[u8]

&[u8] -> &str

```rust
let a = b"version";

let X:&str = std::str::from_utf8(a).unwrap();
```

&[u8] -> String

```rust
let a = b"version";

let X: String = String::from_utf8(a.to_vec()).unwrap();
```

&[u8] -> Vec<u8>

```rust
let a = b"version";

let X:Vec<u8> =a.to_vec();
```
## Vec<u8>

Vec<u8> -> &str

```rust
let a: Vec<u8> = "version".as_bytes().to_vec();

let X:&str = std::str::from_utf8(&a).unwrap();
```

Vec<u8> -> String

```rust
let a: Vec<u8> = "version".as_bytes().to_vec();

let X:String = String::from_utf8(&a).unwrap();
```

Vec<u8> -> &[u8]

```rust
let a: Vec<u8> = "version".as_bytes().to_vec();

let X: &[u8] = &a;
let Y: &[u8] = a.as_slice();
```