---
title: "压缩Go可执行程序文件体积"
date: 2019-06-21T17:43:11+08:00
categories:
- Go
tags:
- Go
keywords:
- Go
---

`go`语言程序默认使用静态编译，生成的可执行程序不依赖任何动态链接库，可以任意部署到各种运行环境，不用担心依赖库的版本问题。

<!--more-->

因为`go`语言是静态编译的，而`C`的编译（比如gcc编译器）都是动态链接库形式编译的，所以导致了`go`生成的可执行文件比`C`语言生的可执行程序稍微大一点的问题。

解决这个问题的办法如下：

执行`go build` 时加上`-ldflags "-s -w"`

* `-s`的作用是去掉符号信息
* `-w`的作用是去掉调试信息


## 参考链接

[Shrink your Go binaries with this one weird trick](https://blog.filippo.io/shrink-your-go-binaries-with-this-one-weird-trick/)
