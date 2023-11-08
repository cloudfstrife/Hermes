---
title: "隐藏Go语言可执行程序中的源码路径"
date: 2021-06-25T10:15:41+08:00
categories:
- Go
tags:
- godoc
- trimpath
keywords:
- godoc
- trimpath
---

Go 语言编译后的可执行程序中，一般包含源码的一些信息，当panic的时候，会暴露出编译时的源码路径，做为一个强迫症晚期患者，这是不可容忍的。Go语言编译器提供了选项 `-trimpath` 用于解决这个问题。

<!--more-->

## 示例代码

```go
package main

import (
	"github.com/nectarian/log"
)

func main() {
	log.Info("go")

	log.Panic("== WARNING ==")
}
```

## 运行

```text
$ go build -o testing
$ ./testing
2021-06-25T10:21:40+08:00	info	testing/main.go:8	go
2021-06-25T10:21:40+08:00	panic	testing/main.go:10	== WARNING ==
panic: == WARNING ==

goroutine 1 [running]:
go.uber.org/zap/zapcore.(*CheckedEntry).Write(0xc00015e0c0, 0x0, 0x0, 0x0)
	/path/to/go_path/pkg/mod/go.uber.org/zap@v1.17.0/zapcore/entry.go:234 +0x58d
go.uber.org/zap.(*Logger).Panic(0xc0000744b0, 0x601fd1, 0xd, 0x0, 0x0, 0x0)
	/path/to/go_path/pkg/mod/go.uber.org/zap@v1.17.0/logger.go:227 +0x85
github.com/nectarian/log.Panic(...)
	/path/to/go_path/pkg/mod/github.com/nectarian/log@v0.0.0-20210621105621-28566f320581/log.go:135
main.main()
	/path/to/source/main.go:10 +0x8d
```

> 上面的 panic 信息中包含了源代码路径和 `GOPATH` 环境变量的路径。

## 使用trimpath

```text
$ go build -o testing -trimpath
$ ./dist/testing/testing 
Version
--------------------------------------------------------------
Version:        v0.0.1
Commit Hash:    
Build Time:     2021-06-24 17:27:07
--------------------------------------------------------------
2021-06-25T10:24:06+08:00	info	testing/main.go:12	go
2021-06-25T10:24:06+08:00	panic	testing/main.go:14	== WARNING ==
panic: == WARNING ==

goroutine 1 [running]:
go.uber.org/zap/zapcore.(*CheckedEntry).Write(0xc00015e0c0, 0x0, 0x0, 0x0)
	go.uber.org/zap@v1.17.0/zapcore/entry.go:234 +0x76d
go.uber.org/zap.(*Logger).Panic(0xc00011a460, 0x6f35bb, 0xd, 0x0, 0x0, 0x0)
	go.uber.org/zap@v1.17.0/logger.go:227 +0x9b
github.com/nectarian/log.Panic(...)
	github.com/nectarian/log@v0.0.0-20210621105621-28566f320581/log.go:135
main.main()
	app/testing/main.go:14 +0xc5
```

> 此时 panic 信息中的源码路径已经是相对路径了