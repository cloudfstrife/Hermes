---
title: "time - 日期与时间格式化"
date: 2023-06-16T15:12:16+08:00
lastmod: 2023-06-16T15:12:16+08:00

categories:
  - Rust
tags:
  - rust
  - time
keywords: 
  - rust
  - time

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Rust 默认自带的 std 库中没有日期与时间格式化功能，[time](https://crates.io/crates/time) crate 增强了 Rust 对于日期与时间的处理，增加了日期时间格式化与解析的能力。

<!--more-->

## 

```console
cargo add time --features formatting,macros,parsing
```

## 创建日期与时间 

创建日期 

```rust
use time::macros::date;
use time::{Date, Month};

fn main() {
    // 使用宏创建
    let x = date!(2022 - 01 - 02);

    // 使用函数创建
    let x = Date::from_calendar_date(2022, Month::January, 2).unwrap();
}
```

创建时间

```rust
use time::macros::time;
use time::Time;

fn main() {
    // 使用宏创建
    let x = time!(0:0:0.0);
    // 使用函数创建
    let x = Time::from_hms(0, 0, 0).unwrap();
    let x = Time::from_hms_micro(0, 0, 0, 0).unwrap();
    let x = Time::from_hms_milli(0, 0, 0, 0).unwrap();
    let x = Time::from_hms_nano(0, 0, 0, 0).unwrap();
}
```

创建日期与时间

`PrimitiveDateTime` 类型包含时间与时间 

```rust
use time::macros::datetime;
use time::Date;

fn main() {
    // 使用宏创建
    let a = datetime!(2023-01-01 0:0:0.0);

    // 为日期添加时间部分
    let u = Date::from_calendar_date(2023, time::Month::January, 1).unwrap();
    let v = u.with_hms(0, 0, 0).unwrap();
    let w = u.with_hms_micro(0, 0, 0, 0).unwrap();
    let x = u.with_hms_milli(0, 0, 0, 0).unwrap();
    let y = u.with_hms_nano(0, 0, 0, 0).unwrap();
}
```

创建带时区的日期与时间 

`OffsetDateTime` 类型包含日期，时间， UTC offset

```rust
use time::macros::datetime;
use time::macros::offset;
use time::OffsetDateTime;

fn main() {
    // 使用宏创建
    let x = datetime!(2023-01-01 0:0:0.0 UTC);
    let x = datetime!(2022-01-02 11:12:13 +8);
    let x = datetime!(2022-01-02 11:12:13.123_456_789 -2:34:56);

    // 使用函数创建
    let x = OffsetDateTime::now_utc();

    let x = OffsetDateTime::from_unix_timestamp(0);
    let x = OffsetDateTime::from_unix_timestamp_nanos(0);

    // 为 PrimitiveDateTime 添加时区

    let x = datetime!(2023-01-01 0:0:0.0);

    // let x = x.assume_utc();
    // let x = x.assume_offset(UtcOffset::from_hms(1, 2, 3));

    // with macros:
    let _ = x.assume_offset(offset!(-11));
}
```

## 格式化输出 

格式化组件的说明文档 [Format description](https://time-rs.github.io/book/api/format-description.html)

```rust
use time::format_description;
use time::macros::datetime;

fn main() {
    let x = datetime!(2022-01-02 11:12:13 +8);
    let format =
        format_description::parse("[year]-[month]-[day] [hour]:[minute]:[second] [offset_hour]:[offset_minute]:[offset_second]").unwrap();

    let result = x.format(format.as_slice()).unwrap();
    println!("{}", result)
}
```

## 日期解析

```rust
use time::format_description;
use time::OffsetDateTime;

fn main() {
    let x = "2022-01-02 11:12:13 08:00:00";
    let format =
        format_description::parse("[year]-[month]-[day] [hour]:[minute]:[second] [offset_hour]:[offset_minute]:[offset_second]").unwrap();

    let result = OffsetDateTime::parse(x, format.as_slice()).unwrap();
    println!("{}", result)
}
```

## 日期运算

```rust
use time::macros::datetime;
use time::Duration;

fn main() {
    let a = datetime!(2022-01-01 10:00:55);
    let b = datetime!(2022-01-01 13:00:00);

    let duration: Duration = b - a;

    println!("{}", duration);
}
```