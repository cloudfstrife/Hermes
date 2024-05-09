---
title: "Rust迭代器"
date: 2023-03-30T14:17:38+08:00
lastmod: 2023-03-30T14:17:38+08:00

categories:
  - Rust
tags:
  - Rust
  - Iterator
keywords: 
  - Rust
  - Iterator

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Rust 迭代器允许应用程序迭代一个连续的集合，例如数组、Vec、HashMap 等。在此过程中，应用程序只需关心如何处理集合中的元素，无需关心如何开始迭代、什么时候结束迭代、如何访问元素等问题。

迭代器适配器会消费迭代器中的元素，并返回一个新的迭代器。迭代器适配器是惰性的，意味着需要一个消费者适配器来收尾，最终将迭代器转换成一个具体的值。

消费适配器是迭代器上的方法，它会消费掉迭代器中的元素，然后返回某一类型的值。

<!--more-->

## 生成迭代器

迭代器可以通过 `iter` `into_iter` `iter_mut` 函数从标准库的大多数集合中获得。

### into_iter、iter、iter_mut 的区别

* into_iter 会夺走所有权
* iter 是借用
* iter_mut 是可变借用


## 迭代适配器

### filter

`filter` 过滤，如果闭包的调用返回 true ，则放入返回的迭代器中

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    let i:i32 = x.filter(|x| x % 2 == 0).sum();
    println!("{}", i);
}
```

### inspect 

`inspect` 用于检查迭代器中的元素，不对迭代器中的元素进行修改

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    let i: i32 = x.inspect(|x| println!("i = {}", x)).sum();
    println!("{}", i);
}
```

### map

`map` 用于将一个迭代器的元素映射到返回的迭代器，常用于元素的类型转换

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    let i: Vec<String> = x.map(|x| x.to_string()).collect();
    println!("{:?}", i);
}
```

### filter_map

`filter_map` 是 `filter` 与 `map` 的结合，闭包返回 Option ， 返回值为 Some 的元素会被加入到返回的迭代器中  

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();

    let i: Vec<String> = x
        .filter_map(|v| {
            if v % 2 == 0 {
                Some(v.to_string())
            } else {
                None
            }
        })
        .collect();

    println!("{:?}", i);
}
```

### chain

`chain` 用于合并了两个迭代器

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    let y = vec![9, 8, 7, 6, 5, 4, 3, 2, 1].into_iter();

    let z: Vec<i32> = x.chain(y).collect();
    println!("{:?}", z)
}
```

### flatten

`flatten` 用于将多层嵌套的集合，整合成一个单层的集合。 

```rust
fn main() {
    let x = vec![vec![1, 2, 3, 4, 5], vec![6, 7, 8, 9, 10]].into_iter();
    let z: Vec<u64> = x.flatten().collect();

    println!("{:?}", z)
}
```

### zip 

`zip` 的作用就是将两个迭代器的内容压缩到一起，形成 `Iterator<Item=(ValueFromA, ValueFromB)>` 这样的新的迭代器

```rust
use std::collections::HashMap;

fn main() {
    let x = vec![1, 3, 5, 7, 9, 11];
    let y = vec![2, 4, 6, 8, 10];
    let z: HashMap<u64, u64> = x.into_iter().zip(y).collect();

    println!("{:?}", z)
}
```

### enumerate

`enumerate` 的作用是使用迭代元素的下标和元素构建新的迭代器，新的迭代器类似于 `Iterator<Item=(index, value)>`

```rust
fn main() {
    let x = vec![1, 3, 5, 7, 9, 11];

    for (i, v) in x.iter().enumerate() {
        println!("{}: {}", i, v)
    }
}
```


## 消费适配器

### for .. in .. 

`for .. in ..` 循环是最简单的消费适配器

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    for i in x {
        print!("{} ", i);
    }
    println!();
}
```

### for_each

`for_each` 与 `for .. in ..` 循环类似

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    x.for_each(|i| print!("{} ", i));
    println!();
}
```

### collect

`collect` 可以用来将一个迭代器转化为目标集合类型

```rust
use std::collections::HashSet;

fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    let set: HashSet<i32> = x.collect();

    for i in set.iter() {
        print!("{} ", i);
    }
    println!();
}
```

### reduce

`reduce` 用于对集合元素累积操作

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    let i = x
        .reduce(|x, y| {
            println!("{} {}", x, y);
            x + y
        })
        .unwrap();

    println!("{}", i);
}
```

### fold

`fold` 与 `reduce` 类似，但可以返回一个与迭代器项不同类型。

```rust
fn main() {
    let x = vec![1, 2, 3, 4, 5, 6, 7, 8, 9].into_iter();
    let i = x.fold(String::new(), |x, y| format!("{} {}", x, y));

    println!("{}", i);
}
```
