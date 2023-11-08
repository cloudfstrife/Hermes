---
title: "Go语言内部包(Internal)"
date: 2019-10-09T18:13:22+08:00
categories:
- Go
tags:
- Go
- internal
keywords:
- Go
- internal
---

Go语言1.4版本增加了 [Internal packages](https://golang.google.cn/doc/go1.4#internalpackages) 特征用于控制包的导入，即internal package只能被特定的包导入。

<!--more-->

内部包的规范约定：导出路径包含`internal`关键字的包，只允许`internal`的父级目录及父级目录的子包导入，其它包无法导入。

## 示例

```text
.
|-- checker
|   |-- internal
|   |   |-- cpu
|   |   |   `-- cpu.go
|   |   `-- ram
|   |       `-- ram.go
|   `-- server.go
|-- go.mod
|-- go.sum
`-- main.go

```

如上包结构的程序，`checker/internal/cpu`和`checker/internal/ram`只能被`checker`包及其子包中的代码导入，不能被`main.go`导入。当在`main.go`中导入并调用其函数，编译期会报如下错误：

```text
$ go build
main.go:10:2: use of internal package app/testing/checker/internal/cpu not allowed

```

