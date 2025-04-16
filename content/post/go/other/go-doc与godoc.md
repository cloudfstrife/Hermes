---
title: "go doc 与 godoc"
date: 2019-11-19T12:13:52+08:00
categories:
- Go
tags:
- Go
keywords:
- Go
---

Go语言项目十分重视代码的文档，在软件设计中，文档对于软件的可维护和易使用具有重大的影响。因此，文档必须是书写良好并准确的，与此同时它还需要易于书写和维护。

<!--more-->

## Go语言注释

Go语言中注释一般分为两种，分别是**单行注释**和**多行注释**

* 单行注释是以 `//` 开头的注释，可以出现在任何地方。
* 多行注释也叫块注释，以 `/*` 开头，以 `*/` 结尾，不可以嵌套使用，一般用于包的文档描述或注释成块的代码片段。

每一个 `package` 都应该有相关注释，在 `package` 语句之前的注释内容将被默认认为是这个包的文档， `package` 的注释应该提供一些相关信息并对整体功能做简要的介绍。

在日常开发过程中，可以使用`go doc`和`godoc`命令生成代码的文档。

## go doc

`go doc` 命令打印Go语言程序实体上的文档。可以使用参数来指定程序实体的标识符。

> **Go语言程序实体**是指变量、常量、函数、结构体以及接口。
>
> **程序实体标识符**就是程序实体的名称。

### go doc 用法

```plaintext
go doc [-u] [-c] [package|[package.]symbol[.methodOrField]]
```

可用的标识：

| 标识 | 说明                                                                  |
| ---- | --------------------------------------------------------------------- |
| -all | 显示所有文档                                                          |
| -c   | 匹配程序实体时，大小写敏感                                            |
| -cmd | 将命令（main包）视为常规程序包，如果要显示main包的doc，请指定这个标识 |
| -src | 显示完整源代码                                                        |
| -u   | 显示未导出的程序实体                                                  |

**示例**

输出指定 package ，指定类型，指定方法的注释

```plaintext
$ go doc sync.WaitGroup.Add
```

输出指定 package ，指定类型的所有程序实体，包括未导出的

```plaintext
$ go doc -u -all sync.WaitGroup
```

输出指定 package 的所有程序实体（非所有详细注释）

```plaintext
$ go doc -u sync
```

## godoc

`godoc`命令主要用于在无法联网的环境下，以web形式，查看Go语言标准库和项目依赖库的文档。

在 `go 1.12` 之后的版本中，`godoc`不再做为go编译器的一部分存在。依然可以通过`go get`命令安装：

```plaintext
go get -u -v golang.org/x/tools/cmd/godoc
```

国内的安装方法

```plaintext
mkdir -p $GOPATH/src/golang.org/x
cd $GOPATH/src/golang.org/x
git clone https://github.com/golang/tools.git
cd tools/cmd/godoc
go install 
ls -alh $GOPATH/bin
```

`godoc` 的帮助信息

```plaintext
$ godoc --help
usage: godoc -http=localhost:6060
  -analysis string
        comma-separated list of analyses to perform when in GOPATH mode (supported: type, pointer). See https://golang.org/lib/godoc/analysis/help.html
  -goroot string
        Go root directory (default "/xxx/xxx")
  -http string
        HTTP service address (default "localhost:6060")
  -index
        enable search index
  -index_files string
        glob pattern specifying index files; if not empty, the index is read from these files in sorted order
  -index_interval duration
        interval of indexing; 0 for default (5m), negative to only index once at startup
  -index_throttle float
        index throttle value; 0.0 = no time allocated, 1.0 = full throttle (default 0.75)
  -links
        link identifiers to their declarations (default true)
  -maxresults int
        maximum number of full text search results shown (default 10000)
  -notes string
        regular expression matching note markers to show (default "BUG")
  -play
        enable playground
  -templates string
        load templates/JS/CSS from disk in this directory
  -timestamps
        show timestamps with directory listings
  -url string
        print HTML for named URL
  -v    verbose mode
  -write_index
        write index to a file; the file name must be specified with -index_files
  -zip string
        zip file providing the file system to serve; disabled if empty

```

`godoc` 有两种运行模式，不指定 `-http` 标记，以命令行模式运行，与`go doc`差不多；当指定 `-http` 标记时，可以通过浏览器查看标准库和第三方库的文档。

```plaintext
godoc -http=:6060
```

打开浏览器，访问 [http://127.0.0.1:6060](http://127.0.0.1:6060) ，点击上方的 `Packages` 可以查阅代码的文档。

### GOPATH模式与module模式

如果执行以上命令的目录下 **没有** `go.mod` 文件，将以 **GOPATH模式** 运行。

```plaintext
$ godoc -http=:6060
using GOPATH mode
```

此模式运行时，`Packages`页面的 `Third party` 显示的是 `$GOPATH/src` 下的`package`包。

如果执行以上命令的目录下 **有** `go.mod` 文件，将以 **module模式** 运行。

```plaintext
godoc -http=:6060
using module mode; GOMOD=/xxx/xxxx/xxx/go.mod
```

此模式运行时，`Packages`页面的 `Third party` 显示的是当前项目所依赖的所有`package`。
