---
title: "hyper - 底层HTTP实现"
date: 2023-04-03T10:11:12+08:00
lastmod: 2023-04-03T10:11:12+08:00

categories:
  - Rust
tags:
  - Rust
  - Hyper
  - HTTP
keywords: 
  - Rust
  - Hyper
  - HTTP

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

[hyper](https://github.com/hyperium/hyper) 是一个用Rust编写的 HTTP 实现，支持HTTP/1和HTTP/2，支持异步Rust，并且同时提供了服务端和客户端的API支持。

<!--more-->

## 服务端

### 添加依赖

```text
$ cargo add hyper --features full
$ cargo add tokio --features full
```

### 服务端

```rust
use hyper::service::{make_service_fn, service_fn};
use hyper::{Body, Request, Response, Server};
use hyper::{Method, StatusCode};
use std::convert::Infallible;
use std::net::SocketAddr;
use std::str::FromStr;

// service 函数，使用这个函数实现处理请求并返回响应的逻辑
async fn route(req: Request<Body>) -> Result<Response<Body>, Infallible> {
    let mut response = Response::new(Body::empty());

    match (req.method(), req.uri().path()) {
        (&Method::GET, "/") => {
            *response.body_mut() = Body::from("Try POSTing data to /echo");
        }
        (&Method::POST, "/echo") => {
            *response.body_mut() = req.into_body();
        }
        _ => {
            *response.status_mut() = StatusCode::NOT_FOUND;
        }
    };

    Ok(response)
}

#[tokio::main]
async fn main() {
    // 监听 8080 端口
    let addr = SocketAddr::from_str("127.0.0.1:8080").unwrap();

    // 每个连接都需要一个处理函数，make_service_fn 生成 AddrStream 的处理函数 
    let make_svc = make_service_fn(|_conn| async {
        // service_fn 将函数转换为 service 方法 
        Ok::<_, Infallible>(service_fn(route))
    });

    let server = Server::bind(&addr).serve(make_svc);

    if let Err(e) = server.await {
        eprintln!("server error: {}", e);
    }
}
```

## 客户端

```rust
use hyper::body::HttpBody;
use hyper::Body;
use hyper::Client;
use hyper::Method;
use hyper::Request;

#[tokio::main]
async fn main() {
    let client = Client::new();
    let request = Request::builder()
        .method(Method::POST)
        .uri("http://127.0.0.1:8080/echo")
        .body(Body::from("version"))
        .unwrap();
    let mut response = client.request(request).await.unwrap();

    let body = response.body_mut();
    let data = body.data().await.unwrap().unwrap();
    println!("{}", data.escape_ascii());
}
```

#### TLS 支持

添加 hyper-tls 依赖

```text
$ cargo add hyper-tls
```

实现

```rust
use hyper::body::HttpBody;
use hyper::Body;
use hyper::Client;
use hyper::Method;
use hyper::Request;
use hyper_tls::HttpsConnector;

#[tokio::main]
async fn main() {
    // 创建 TLS HttpsConnector 
    let tlscon = HttpsConnector::new();
    // 使用 TLS HttpsConnector 构建 Client
    let client = Client::builder().build(tlscon);

    let request = Request::builder()
        .method(Method::GET)
        .uri("https://cn.bing.com")
        .body(Body::from(""))
        .unwrap();
    let mut response = client.request(request).await.unwrap();

    let body = response.body_mut();
    let data = body.data().await.unwrap().unwrap();
    println!("{}", data.escape_ascii());
}
```