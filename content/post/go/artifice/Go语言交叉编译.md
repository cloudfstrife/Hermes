---
title: "Go语言交叉编译"
date: 2019-11-07T18:08:45+08:00
categories:
- Go
tags:
- Go
- 交叉编译
keywords:
- Go
- 交叉编译
---

交叉编译是指在一种计算机环境中，使用支持交叉编译的编译器，将源代码编译成可以运行在另一种计算机环境的可执行程序的过程。

Go语支持交叉编译，可以在一个平台上生成另一个平台的可执行程序。

<!--more-->

## Go语言交叉编译

要使用Go语言的交叉编译功能，只需要两步：

* 设置`GOOS`和`GOARCH`环境变量为目标环境的操作系统和架构。
* 执行编译操作

**示例** 

在Linux操作系统，编译arm架构 Linux操作系统的可执行程序

```text
GOOS=linux GOARCH=arm go build -v 
```

在Windows操作系统，编译amd64架构 freebsd操作系统可执行程序

```text
set GOOS=freebsd

set GOARCH=amd64

go build -v
```

平台相关的环境变量

```text
$GO386
$GOARM
$GOMIPS
$GOMIPS64 
$GOPPC64
```

有效的 `$GOOS`,`$GOARCH` 组合以及平台相关环境变量

参见[https://golang.org/doc/install/source#environment](https://golang.org/doc/install/source#environment) 

国内可访问： [https://golang.google.cn/doc/install/source#environment](https://golang.google.cn/doc/install/source#environment)
