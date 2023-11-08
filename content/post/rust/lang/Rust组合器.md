---
title: "Rust组合器"
date: 2023-03-29T14:15:07+08:00
categories:
- Rust
tags:
- Rust
- Combinators
keywords:
- rust
- Combinators
---

```text
Combinators are methods that ease the manipulation of some type T . They favor a functional (method chaining) style of code.
                                                                        -- 《Black Hat Rust》 Sylvain Kerkour
```

组合器是 *简化* 对某些类型T的操作的方法。它提倡以函数式（方法链）风格编写代码。 

本文列举一下常见的组合器，可用于 `Option`、`Result` 等类型。

<!--more-->

## Option 常用 Combinators

### or 

如果 Option 的值不为 None ，则直接返回此 Option ，如果 Option 的值是 None ，则返回 or 函数的参数 ( Option 类型)。

```rust
fn main() {
    let port: Option<String> = std::env::var("PORT").ok();
    let port = port.or(Some(String::from("8080")));
    println!("{}", port.unwrap());
}

// 输出
// 8080
```

### or_else

与 or 类似，只是参数是一个返回 Option 类型的闭包

```rust 
fn main() {
    let port: Option<String> = std::env::var("PORT").ok();
    let port = port.or_else(|| Some(String::from("8080")));
    println!("{}", port.unwrap());
}

// 输出
// 8080
```

### and 和 and_then

如果调用 and 或者 and_then 的值为 None 则直接返回 None，否则返回参数中的内容，区别在于 and 的参数是 Option 类型的值，and_then 函数的参数是一个闭包，接收 Option *包裹的类型*，并返回一个 Option

```rust
fn main() {
    let a: Option<String> = Some(String::from("8080"));
    let b: Option<String> = Some(String::from("9090"));
    let c: Option<String> = a.and(b);

    let x: Option<String> = None;
    let y: Option<String> = Some(String::from("8080"));

    let z = y.and(x);

    println!("{:?} , {:?}", c, z);
}

// 输出
// Some("9090") , None

fn main() {
    let a: Option<String> = Some(String::from("8080"));
    let b: Option<String> = a.and_then(|v| Some(String::from(format!("127.0.0.1:{}", v))));

    let x: Option<String> = None;
    let z = x.and_then(|v| Some(v));

    println!("{:?} , {:?}", b, z);
}

// 输出
// Some("127.0.0.1:8080") , None
```

### filter

filter 使用闭包参数对 Option 包裹的数据进行过滤，如果闭包调用返回 false 则整个函数返回 None

```rust
fn main() {
    let port: Option<String> = std::env::var("PORT").ok();
    let l = port.filter(|x| x.len() >= 3);

    println!("{:?}", l);
}


// 输出
// $ ./target/debug/testing
// None
// $ export PORT=90
// $ ./target/debug/testing
// None
// $ export PORT=9090      
// $ ./target/debug/testing
// Some("9090")
```

### map

如果 Option 不为 None，则使用参数的闭包函数转换 Option 包裹的类型

```rust
fn main() {
    let port: Option<String> = std::env::var("PORT").ok();
    let x = port.map(|v| v.parse::<i32>().unwrap());
    println!("{:?}", x);
}

// 输出
// $ ./target/debug/testing
// None
// $ export PORT=9090
// $ ./target/debug/testing
// Some(9090)
// $ unset PORT
```

### map_or

如果 Option 为 None ，则使用默认值，否则，调用闭包转换 Option 包裹的值

```rust
fn main() {
    let port: Option<String> = std::env::var("PORT").ok();
    let x = port.map_or(String::from("127.0.0.1:8080"), |v| {
        format!("127.0.0.1:{}", v)
    });
    println!("{:?}", x);
}

// 输出
// $ ./target/debug/testing
// "127.0.0.1:8080"
// $ export PORT=9090      
// $ ./target/debug/testing
// "127.0.0.1:9090"
// $ unset PORT            
```

### map_or_else

map_or_else 在 Option 为 None 时，调用第一个闭包参数，生成默认值，在 Option 不为 None 时，调用第二个闭包参数对 Some 包裹的数据进行转换


```rust
fn main() {
    let port: Option<String> = std::env::var("PORT").ok();
    let x = port.map_or_else(
        || String::from("127.0.0.1:8080"),
        |v| format!("127.0.0.1:{}", v),
    );
    println!("{:?}", x);
}

// 输出
// $ ./target/debug/testing
// "127.0.0.1:8080"
// $ export PORT=9090
// $ ./target/debug/testing
// "127.0.0.1:9090"
// $ unset PORT
```


### 其它方法

#### unwrap_or

获取 Option 包裹的值，如果 Option 的值是 None ，则返回 unwrap_or 函数的参数。

```rust
fn main() {
    let port:Option<String> =std::env::var("PORT").ok();
    let port = port.unwrap_or(String::from("8080"));
    println!("{}", port);
}

// 输出
// 8080
```

#### is_some 和 is_none

```rust
fn main() {
    let a: Option<u32> = Some(1);
    let b: Option<u32> = None;
    println!("a : {} {} ", a.is_some(), a.is_none());
    println!("b : {} {} ", b.is_some(), b.is_none());
}

// 输出
// a : true false 
// b : false true 
```

## Result 常用 Combinators

### ok

将 Result 转换成 Option 

```rust
fn main() {
    let x: Result<String, String> = Ok("8080".to_string());
    let y: Result<String, String> = Err("No".to_string());

    println!("{:?} {:?}", x.ok(), y.ok())
}

// 输出
// Some("8080") None
```

### or

与 Option 的 or 类似

```rust
fn main() {
    let port: Result<String, std::env::VarError> = std::env::var("PORT");
    let port: Result<String, std::env::VarError> = port.or(Ok(String::from("8080")));
    println!("{}", port.unwrap());
}

// 输出
// 8080
```

### or_else

与 Option 的 or_else 类似，区别在于传入参数的闭包接收一个 Error ，在闭包中处理 Error 并返回新的 Result。

```rust
fn main() {
    let port: Result<String, std::env::VarError> = std::env::var("PORT");
    let port: Result<String, std::env::VarError> = port.or_else(|x| {
        println!("{}", x);
        Ok(String::from("8080"))
    });
    println!("{}", port.unwrap());
}

// 输出
// environment variable not found
// 8080
```

### and

与 Option 的 and 函数类似

```rust
fn main() {
    let port: Result<String, std::env::VarError> = std::env::var("PORT");
    let port: Result<String, std::env::VarError> = port.and(Ok(String::from("9090")));
    println!("{:?}", port);
}


// 输出 
// $ ./target/debug/testing
// Err(NotPresent)
// $ export PORT=9090
// $ ./target/debug/testing
// Ok("9090")
// $ unset PORT
```

### and_then

与 Option 的 and_then 函数类似，区别在于传入参数的闭包接收一个Result包裹的类型参数 ，在闭包中可以使用此值来生成新的 Result 值。

```rust
fn main() {
    let port: Result<String, std::env::VarError> = std::env::var("PORT");
    let port: Result<String, std::env::VarError> = port.and_then(|x| {
        println!("{}", x);
        Ok(String::from("8080"))
    });
    println!("{:?}", port);
}

// 输出
// $ ./target/debug/testing
// Err(NotPresent)
// $ export PORT=9090
// $ ./target/debug/testing
// 9090
// Ok("8080")
// $ unset PORT 
```

### map 

与 Option 的 map 类似 

```rust
fn main() {
    let port: Result<String,std::env::VarError> = std::env::var("PORT");
    let x = port.map(|v| v.parse::<i32>().unwrap());
    println!("{:?}", x);
}

// 输出
// $ ./target/debug/testing
// Err(NotPresent)
// $ export PORT=9090
// $ ./target/debug/testing
// Ok(9090)
// $ unset PORT
```

### map_err

当 Result 的值为 Err 时，使用此函数的闭包参数，将 Err 转换，生成另一个 Err 

```rust
fn main() {
    let port: Result<String, std::env::VarError> = std::env::var("PORT");
    let x = port.map_err(|v| {
        println!("{}", v);
        "no found"
    });
    println!("{:?}", x);
}


// 输出
// $ ./target/debug/testing
// environment variable not found
// Err("no found")
// $ export PORT=9090      
// $ ./target/debug/testing
// Ok("9090")
// $ unset PORT
```
### map_or

如果 Result 为 Err ，则使用默认值，否则，调用闭包转换 Result 包裹的值

```rust
fn main() {
    let port: Result<String, std::env::VarError> = std::env::var("PORT");
    let x = port.map_or(String::from("127.0.0.1:8080"), |v| {
        format!("127.0.0.1:{}", v)
    });
    println!("{:?}", x);
}

// 输出
// $ ./target/debug/testing
// "127.0.0.1:8080"
// $ export PORT=9090      
// $ ./target/debug/testing
// "127.0.0.1:9090"
// $ unset PORT            
```

### map_or_else

map_or_else 在 Result 为 Err 时，调用第一个闭包参数处理异常，并生成默认值，在 Result 为 Ok 时，调用第二个闭包，转换 Ok 包裹的数据

```rust
fn main() {
    let port: Result<String, std::env::VarError> = std::env::var("PORT");
    let x = port.map_or_else(
        |err| {
            println!("{}", err);
            String::from("127.0.0.1:8080")
        },
        |v| format!("127.0.0.1:{}", v),
    );
    println!("{:?}", x);
}

// 输出
// $ ./target/debug/testing
// environment variable not found
// "127.0.0.1:8080"
// $ export PORT=9090      
// $ ./target/debug/testing
// "127.0.0.1:9090"
// $ unset PORT            
```


## 参考资料

[Combinators](https://learning-rust.github.io/docs/combinators/)