---
title: "Log4rs 日志"
date: 2025-02-27T15:35:04+08:00
lastmod: 2025-02-27T15:35:04+08:00

categories:
  - Rust
tags:
  - Rust
  - log4rs
  - log
keywords: 
  - Rust
  - log4rs
  - log

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

[log4rs](https://crates.io/crates/log4rs) 是一个高效的日志库框架，借鉴了Java的Logback和log4j。提供了输出到文件和控制台两种方式。

log4rs支持两种配置方式，基于编程方式配置和基于YAML文件配置。 建议使用YAML文件配置，这有效减少重新编译次数。

<!--more-->

示例

## 添加依赖


```bash
cargo add log4rs
```
## 创建配置文件

```bash
mkdir -p config
touch config/log4rs.yaml
```

## 代码

config/log4rs.yaml

```yaml
refresh_rate: 30 seconds
appenders:
  stdout:
    kind: console
  requests:
    kind: file
    path: "log/requests.log"
    encoder:
      pattern: "{d} - {m}{n}"
root:
  level: warn
  appenders:
    - stdout
loggers:
  app::backend::db:
    level: info
  app::requests:
    level: info
    appenders:
      - requests
    additive: false
```

main.rs

```rust
use log::{debug, error, info, trace, warn};
use log4rs;

fn main() {
    log4rs::init_file("config/log4rs.yaml", Default::default()).unwrap();

    error!("booting up");
    warn!("booting up");
    info!("booting up");
    debug!("booting up");
    trace!("booting up");
}
```

## 配置示例

```yaml
refresh_rate: 10 seconds
appenders:
  stdout:
    kind: console
    encoder:
      kind: pattern
      pattern: "{d(%F %T.%6f)} {h({l})} {f}:{L} {m}{n}"
    target: stdout
    tty_only: false
  app_file:
    kind: file
    encoder:
      kind: pattern
      pattern: "{d(%F %T.%6f)} {h({l})} {f}:{L} {m}{n}"
    path: $ENV{PWD}/log/app.log
    append: true
  app_rolling_file_size:
    kind: rolling_file
    encoder:
      kind: pattern
      pattern: "{d(%F %T.%6f)} {h({l})} {f}:{L} {m}{n}"
    path: "$ENV{PWD}/log/app_rolling_size.log"
    policy:
      kind: compound
      trigger:
        kind: size
        limit: 1mb
      roller:
        kind: fixed_window
        base: 1
        count: 5
        pattern: "$ENV{PWD}/log/app_rolling_size.{}.log"
  app_rolling_file_time:
    kind: rolling_file
    path: "$ENV{PWD}/log/app_rolling_time.log"
    policy:
      kind: compound
      trigger:
        kind: time
        interval: 1 day
        modulate: true
        max_random_delay: 0
      roller:
        kind: fixed_window
        base: 1
        count: 5
        pattern: "$ENV{PWD}/log/app_rolling_time.{}.log"
root:
  level: trace
  appenders:
    - stdout
    - app_file
    - app_rolling_file_size
    - app_rolling_file_time
loggers:
  app::http:
    level: info
    appenders:
      - stdout
    additive: false
```

