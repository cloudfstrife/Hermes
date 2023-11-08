---
title: "最简单的cgo示例"
date: 2020-09-28T09:03:37+08:00
categories:
- go
- cgo
tags:
- go
- cgo
keywords:
- go
- cgo
---

本文以一个最简单的 cgo 示例，说明如何封装 C 语言动态链接库，供 Go 程序调用。

<!--more-->

代码结构

```
.
├── say
│   ├── include
│   │   └── say.h
│   ├── lib
│   │   └── libsay.so
│   └── say.c
├── say-go
│   ├── go.mod
│   └── say
│       └── say.go
└── testing
    ├── go.mod
    ├── main.go
    └── testing

```

## say

**include/say.h**

```c
void say(const char* s);
```

**say.c**

```c
#include "say.h"
#include <stdio.h>

void say(const char* s) {
	puts(s);
}
```

**编译**

```text
$ gcc -fPIC -shared -Iinclude say.c -o lib/libsay.so
```

> 注意这里的 `-o lib/libsay.so` 必须以 `lib` 开头，是 Linux C 约定的。

## say-go

**say/say.go**

```go
package say

/*
#cgo LDFLAGS: -lsay
#include "say.h"
*/
import "C"

// Say call c function say(const char* s)
func Say(s string) {
	C.say(C.CString(s))
}
```

> 这里的 `#cgo LDFLAGS: -lsay` 会在链接阶段去动态链接库目录查找 `libsay.so` 文件

**go.mod**

```text
module github.com/cloudfstrife/say-go

go 1.15
```

## testing

**main.go**

```go
package main

import (
	"github.com/cloudfstrife/say-go/say"
)

func main() {
	say.Say("cgo")
}
```


**go.mod**


```text
module app/testing

go 1.15

require github.com/cloudfstrife/say-go v1.0.0

replace github.com/cloudfstrife/say-go v1.0.0 => /source/space/say-go
```

> 因为不想上传github，所以用了 replace ，如果要测试，目录自行替换

**编译**

```text
$ mkdir -p 3rd/include
$ mkdir -p 3rd/lib
$ cp ../say/include/say.h 3rd/include
$ cp ../say/lib/libsay.so 3rd/lib
$ CGO_CFLAGS="-g -O2 -I /source/space/testing/3rd/include" CGO_LDFLAGS="-g -O2 -L /source/space/testing/3rd/lib -Wl,-rpath=./3rd/lib" go build
```

> 这里拷贝了 到当前目录，模拟第三方库只提供
>
> `CGO_CFLAGS="-g -O2 -I /source/space/testing/3rd/include"` 指定编译阶段查找头文件的目录，可以有多个，以冒号 `:` 分隔
> 
> `CGO_LDFLAGS="-g -O2 -L /source/space/testing/3rd/lib -Wl,-rpath=./3rd/lib" ` 指定链接阶段查找动态链接库的目录，可以有多个，以冒号 `:` 分隔
>
> `-Wl,-rpath=./3rd/lib` 指定运行可执行程序时搜索动态链接库的目录，可以有多个，以冒号 `:` 分隔

**运行**

```
$ ./testing
cgo
```
