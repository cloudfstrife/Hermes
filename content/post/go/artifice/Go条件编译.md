---
title: "Go条件编译"
date: 2019-08-21T18:19:16+08:00
categories:
- Go
tags:
- Go
keywords:
- Go
---

当程序的某个功能需要依赖底层平台或者根据特定处理器进行不同的实现时，提供特定的实现就非常有必要。

<!--more-->

C语言可以使用`#define`来控制是否包含平台相关的特定代码。Go语言也提供了**命名约定**和**条件构建标识**两种方式来实现编译特定代码。

## 命名约定

**命令约定**通过源码文件的文件名实现的。  
如果一个`go`源文件的的文件名，去掉扩展名和可能存在的`_test`后缀之后，符合下面的模式，则只在特定操作系统或者架构编译。

* _GOOS
* _GOARCH
* _GOOS_GOARCH

> 其中GOOS和GOARCH分别代表任何已知的操作系统和体系结构值，见`runtime.GOOS`和`runtime.GOARCH`。

示例：

```text
source_windows_amd64.go
```

## 条件构建标识

**条件构建标识**通过代码注释的形式实现。  
条件构建标识以`// +build`开头，必须写在文件的最顶端，前面只能有空白行或者其它行注释，构建标识与`package`之间必须有一个空白行。   
`go build`指令在编译时，检查每个文件的条件构建标识，决定是否编译这个文件。

### 条件构建标识规则

* 不同tag之间以空格分割的，tag与tag之间的关系是OR关系。
* 不同tag之间以逗号分割的，tag与tag之间的关系是AND关系。
* tag由字母和数据区分，以!开头的表示条件"非"
* 当一个文件存在多个tag时，tag与tag之间的关系是AND关系。

示例：

```go
// +build linux,386 darwin,!cgo
```

表示 

```text
(linux AND 386) OR (darwin AND (NOT cgo))
```

示例：

```go
// +build linux darwin
// +build 386
```

表示 

```text
(linux OR darwin) AND 386
```

当一个文件在任何条件下都不被构建时，使用下面的构建标识 

```go
// +build ignore
```

在一次特定构建中，条件构建标识需要满足以下值

- `runtime.GOOS`中对操作系统的定义
- `runtime.GOARCH`中对CPU架构的定义
- 正在使用的编译器: `gc` 或者 `gccgo`
- 如果`ctxt.CgoEnabled`为true时，支持`cgo`
- `go1.1`, 代表Go编译器版本大于1.1时编译
- `go1.2`, 代表Go编译器版本大于1.2时编译
- `go1.3`, 代表Go编译器版本大于1.3时编译
- `go1.4`, 代表Go编译器版本大于1.4时编译
- `go1.5`, 代表Go编译器版本大于1.5时编译
- `go1.6`, 代表Go编译器版本大于1.6时编译
- `go1.7`, 代表Go编译器版本大于1.7时编译
- `go1.8`, 代表Go编译器版本大于1.8时编译
- `go1.9`, 代表Go编译器版本大于1.9时编译
- `go1.10`, 代表Go编译器版本大于1.10时编译
- `go1.11`, 代表Go编译器版本大于1.11时编译
- `go1.12`, 代表Go编译器版本大于1.12时编译
- `ctxt.BuildTags`列出的其它标识

## Binary-Only Packages

当一个包用于存放二进制形式的包，不包含用于编译的源代码时，可以在包中存放一个不被条件编译排除的文件，文件中包含`//go:binary-only-package`注释。和构建标识一样，这个标识必须写在文件的最顶端，前面只能有空白行或者其它行注释。与`package`之间必须有一个空白行。  
与条件构建标识不同的是`//go:binary-only-package`只能出现在非测试文件中。

示例：

```go
//go:binary-only-package

package mypkg
```

上面的源代码可能包含其他Go代码，但是该代码永远不会被编译。

`godoc`这样的工具可以处理这样的代码，可能作为最终用户文档来使用。


## 列出条件编译文件列表

`go list` 子命令用于输出包名，每个一行。最有用的参数是 `-f` 和 `-json`，用于控制每个包的输出格式及内容。

`-f`使用`template`包的语法，指定每个包的输出内容和格式。默认的输出是`{{.ImportPath}}`，传递给模板是`go/build`包中的[Packages](https://golang.org/pkg/go/build/#Package) 类型。

跟据不同的`GOOS`，`go list`输出的GoFiles也不相同。

示例：

```text
$ export GOOS=linux
$ go list -f '{{.GoFiles}}' os/exec
[exec.go exec_unix.go lp_unix.go]
$ export GOOS=windows
$ go list -f '{{.GoFiles}}' os/exec
[exec.go exec_windows.go lp_windows.go]
```
