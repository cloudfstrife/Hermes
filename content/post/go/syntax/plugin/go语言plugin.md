---
title: "Go语言plugin"
date: 2021-04-08T17:57:44+08:00
categories:
- Go
tags:
- Go
- plugin
keywords:
- Go
- plugin
---

Golang是静态编译型语言，在编译时就将所有引用的库全部加载打包到最终的可执行程序中（CGO除外），在运行时不需要动态加载其他共享库。这样设计的好处很多，但是如果某些场景下需要实现功能的可插拔则非常不方便。
在Go 1.8更新中，Go语言提供了Go Plugin机制，可以在运行时动态加载外部功能。

<!--more-->

## Go的 Plugin 实现

Go语言提供了标准库 `plugin` 来实现插件机制。其接口非常简单

```
type Plugin
    func Open(path string) (*Plugin, error)
    func (p *Plugin) Lookup(symName string) (Symbol, error)
type Symbol
```

* **Open** : 用于加载编译好的插件。
* **Lookup** : 搜索symbol。
* **Symbol** : 是指向变量或函数的指针。

## 示例

创建目录结构

```
mkdir hello-plugin world-plugin testing

cd hello-plugin
go mod init app/hello-plugin
cd ../world-plugin
go mod init app/world-plugin
cd ../testing
go mod init app/testing

touch ./{hello-plugin,world-plugin,testing}/mian.go
```

目录结构如下：

```
.
├── hello-plugin
│   ├── go.mod
│   └── mian.go
├── testing
│   ├── go.mod
│   └── mian.go
└── world-plugin
    ├── go.mod
    └── mian.go
```

**hello-plugin/mian.go**

```go
package main

import "fmt"

func Say() {
	fmt.Println("Hello")
}
```

**world-plugin/mian.go**

```go
package main

import "fmt"

func Say() {
	fmt.Println("World")
}
```

**testing/main.go**

```go
package main

import (
	"log"
	"plugin"
)

func main() {
	p, err := plugin.Open("./say.so")
	if err != nil {
		log.Fatalf("error open plugin: %v", err)
	}
	s, err := p.Lookup("Say")
	if err != nil {
		log.Fatalf("error lookup component: %v", err)
	}
	if say, ok := s.(func()); ok {
		say()
	}
}
```

### 编译与构建

```
cd hello-plugin
go build --buildmode=plugin -o say.so
cd ../world-plugin
go build --buildmode=plugin -o say.so
cd ../testing
go build
```

### 测试 

```
cp ../hello-plugin/say.so .
./testing 
rm say.so
cp ../world-plugin/say.so .
./testing 
```

## 总结

在Go语言中使用plugin的基本流程如下

1. 编写插件
2. 编译为so
3. 在主程序中加载插件
4. 搜索可用组件（变量或者函数）
5. 对组件执行类型断言
6. 调用组件

## 参考链接

[plugin - The Go Programming Language](https://golang.google.cn/pkg/plugin/)