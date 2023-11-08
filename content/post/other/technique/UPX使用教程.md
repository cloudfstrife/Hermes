---
title: "UPX使用教程"
date: 2022-10-11T19:59:21+08:00
categories:
- other
- technique
tags:
- upx
keywords:
- upx
---

[UPX](https://upx.github.io/) 是非常优秀的可执行程序压缩的工具，适用于多种不同的可执行格式。UPX 使用 UCL 的压缩算法。UPX 本质是压缩程序代码，减少程序体积，属于压缩壳。

<!--more-->

> UCL : 是一个用 ANSI C 编写的便携式无损数据压缩库，实现了一系列压缩算法，这些算法在实现优秀的压缩比/值的同时，允许非常快速的解压缩，并且解压缩时不需要额外的内存。

## 安装

```text
# Debian 
$ sudo apt install upx-ucl 
```

> [GitHub](https://github.com/upx/upx) 也提供了编译好的各平台的二进制文件

## 使用

### 压缩

```text
$ upx [options] input_file
```

UPX 对文件的默认操作即为压缩，使用上述命令会使用默认参数压缩并替换文件 input_file 。

#### 常用选项

* `-1[23456789]` ： 指定压缩级别，数值越高压缩率越高，耗时越长。对于小于 512 KiB 的文件默认使用 `-8`，其他的默认为 `-7`
* `--best` ： 最高压缩级别
* `--brute` ： 尝试使用各种压缩方式来获取最高压缩比
* `-o [file]` ： 将压缩文件另存为 [file]


### 解压

```text
$ upx -d input_file
```

UPX 的优点

* UPX 可以压缩各种类型的可执行文件
* 压缩后的文件可以直接由操作系统执行
* 压缩过程不会修改源文件，也就意味着解压后直接可以得到原始文件
* 不会产生额外的动态库调用

UPX 的缺点

* 运行的程序不会共享数据段（汇编），所以多实例运行的程序不适合压缩
* 使用 `ldd` 和 `size` 命令无法获取到程序的有效信息

# 原理

UPX 将程序压缩并在头部加入解压的程序，具体的原理可以参考 [Packers, How They Work, Featuring UPX](https://dzone.com/articles/packers-how-they-work-featuring-upx)