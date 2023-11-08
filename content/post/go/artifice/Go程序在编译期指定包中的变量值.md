---
title: "Go程序在编译期指定包中的变量值"
date: 2019-10-31T18:16:11+08:00
categories:
- Go
tags:
- Go
keywords:
- Go
---

Go 语言编译时，可以通过 `-ldflags` 的方式，为指定包中的变量赋值。

<!--more-->

## 格式

```text
$ go build -ldflags "-X '$包名.变量名=$变量值'"
```

## 示例

文件结构

```text
.
|-- go.mod
|-- go.sum
|-- main.go
`-- version
    `-- version.go
```

**go.mod**

```text
module app/testing

go 1.13

require (
        github.com/konsorten/go-windows-terminal-sequences v1.0.2 // indirect
        github.com/sirupsen/logrus v1.4.2
        golang.org/x/sys v0.0.0-20191010194322-b09406accb47 // indirect
)

```

**main.go**

```go
package main

import (
        "app/testing/version"

        log "github.com/sirupsen/logrus"
)

func init() {
        log.SetFormatter(
                &log.TextFormatter{
                        FullTimestamp: true,
                },
        )
}

func main() {
        version.ShowVersion()
}

```

**version/version.go**

```go
package version

import "fmt"

//Version app version
var Version string

//ShowVersion output current version
func ShowVersion() {
        fmt.Println("version: " + Version)
}

```

**编译与运行**

```text
$ go build -ldflags "-X 'app/testing/version.Version=1.0.0'"
$ ./testing.exe
version: 1.0.0
```
