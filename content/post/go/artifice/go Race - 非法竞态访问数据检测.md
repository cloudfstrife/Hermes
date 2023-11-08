---
title: "go race - 非法竞态访问数据检测"
date: 2019-07-31T18:06:51+08:00
categories:
- Go
tags:
- Go
keywords:
- Go
---

**非法竞态访问数据** 是指无任何同步保护下并行读写同一份数据。  
`go`命令内置了非法竞态访问数据的检测工具。可以使用`go run -race`或者`go build -race`来进行竞争检测。

<!--more-->

**示例**

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	a := 1
	go func() {
		a = 2
	}()
	a = 3
	fmt.Println("a is ", a)

	time.Sleep(1 * time.Second)
}
```

运行

```plaintext
$ go run -race main.go
a is  3
==================
WARNING: DATA RACE
Write at 0x00c000064058 by goroutine 6:
  main.main.func1()
      ....../testing/main.go:11 +0x3f

Previous write at 0x00c000064058 by main goroutine:
  main.main()
      ....../testing/main.go:13 +0x8f

Goroutine 6 (running) created at:
  main.main()
      ....../testing/main.go:10 +0x81
==================
Found 1 data race(s)
exit status 66
```

命令输出了Warning，警告`goroutine 6`在11行与`main goroutine`第13行产生了竞态访问，`Goroutine 6`在`main.go`第10行创建。

`-race`会引发CPU和内存的使用增加，所以可以在测试环境使用，不建议在正式环境使用。
