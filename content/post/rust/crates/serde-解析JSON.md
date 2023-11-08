---
title: "serde - Rust解析JSON"
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

JSON 是目前使用最广泛的数据传输格式，在Rust 中，应用程序可以使用 [serde](https://crates.io/crates/serde) 和 [serde-json](https://crates.io/crates/serde_json) 来处理 JSON 的序列化与反序列化。


<!--more-->

## Serde

Serde 是一个 Rust 库，用于序列化和反序列化各种格式（如 JSON，YAML，BSON等）。Serde 提供了一组通用的 API 来支持不同格式，同时还提供了 `#[derive(Serialize, Deserialize)]` 注释来标记需要序列化和反序列化的结构体，然后使用 serde_json 库中的 `to_string` 和 `from_str` 函数来进行 JSON 的序列化和反序列化。

## 添加依赖

```text
$ cargo add serde --features derive
$ cargo add serde_json
```

## 无类型 JSON

如果应用程序不关心 JSON 的数据结构，可以使用 `serde_json` 内置的 `serde_json::Value` 枚举来处理，`serde_json::Value` 的定义如下：

```rust
#[derive(Clone, Eq, PartialEq)]
pub enum Value {
    Null,
    Bool(bool),
    Number(Number),
    String(String),
    Array(Vec<Value>),
    Object(Map<String, Value>),
}
```

```rust
use serde_json::Value;

fn main() {
    let json = r#"
{
    "username": "username-for-you",
    "password": "tdep",
    "orders": [
        {
            "id":1,
            "name": "goods1",
            "count": 10,
            "amount": 10.10
        },
        {
            "id":2,
            "name": "goods2",
            "count": 2,
            "amount": 100.30
        }
    ]
}
"#;
    let parsed: Value = serde_json::from_str(json).unwrap();

    println!("UserName : {:?}", parsed["username"]);
    println!("Password : {:?}", parsed["password"]);
    println!("Orders   : {:?}", parsed["orders"]);
    let orders = parsed["orders"].as_array().unwrap();
    for i in orders {
        println!(
            "id : {} , name : {} , count : {} , amount : {}",
            i["id"], i["name"], i["count"], i["amount"]
        );
    }
}
```

## 序列化与反序列化

`serde` 的 `derive` 特征提供了 `Deserialize` , `Serialize` 的标签，用于标识用于序列化与反序列化的结构

序列化函数

* `serde_json::to_string` 序列化成 String
* `serde_json::to_string_pretty` 序列化成美化的 String
* `serde_json::to_writer` 序列化并写入
* `serde_json::to_vec` 序列化成[u8]
* `serde_json::to_vec_pretty` 序列化成美化的 [u8]

反序列化函数

* `serde_json::from_str` 将 JSON 格式化的字符串序列化成指定类型
* `serde_json::from_reader` 从 reader 中读取数据并反序列化成指定类型
* `serde_json::from_slice` 将 [u8] 反序列化成指定类型


```rust
use serde::{Deserialize, Serialize};

#[derive(Deserialize, Serialize)]
struct User {
    username: String,
    password: String,
    orders: Vec<Order>,
}

#[derive(Deserialize, Serialize)]
struct Order {
    id: u32,
    name: String,
    count: u32,
    amount: f64,
}
fn main() {
    let u = User {
        username: String::from("username-for-you"),
        password: String::from("password-for-you"),
        orders: vec![
            Order {
                id: 1,
                name: String::from("goods1"),
                count: 10,
                amount: 10.1,
            },
            Order {
                id: 2,
                name: String::from("goods1"),
                count: 20,
                amount: 100.3,
            },
        ],
    };

    let x = serde_json::to_string(&u).unwrap();
    println!("{}", x);
}
```



```rust
use serde::{Deserialize, Serialize};

#[derive(Deserialize, Serialize, Debug)]
struct User {
    username: String,
    password: String,
    orders: Vec<Order>,
}

#[derive(Deserialize, Serialize, Debug)]
struct Order {
    id: u32,
    name: String,
    count: u32,
    amount: f64,
}
fn main() {
    let r = r#"{
        "username": "username-for-you",
        "password": "password-for-you",
        "orders": [
          {
            "id": 1,
            "name": "goods1",
            "count": 10,
            "amount": 10.1
          },
          {
            "id": 2,
            "name": "goods1",
            "count": 20,
            "amount": 100.3
          }
        ]
      }
      "#;

    let u = serde_json::from_str::<User>(r).unwrap();
    println!("{:?}", u);
}
```