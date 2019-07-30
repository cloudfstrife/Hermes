---
title: "Go子进程"
date: 2019-07-30T14:05:44+08:00
categories:
- Go
- package
tags:
- Go
- package
- cmd
keywords:
- Go
- package
- 子进程
---

Go语言的`os/exec`包封装了调用外部可执行程序的操作。它包装了`os.StartProcess`，以便更容易映射`stdin`与`stdout`，使用管道连接I/O，并进行其它调整。  
`os/exec`假定运行在Linux环境，windows某些操作可能无法执行，此包的操作也无法在Go Playground上运行。


<!--more-->

## 查找可执行程序

```go
func LookPath(file string) (string, error)
```

**示例**

```go
package main

import (
	"os/exec"

	log "github.com/sirupsen/logrus"
)

func main() {
	s, err := exec.LookPath("ls")
	if err != nil {
		log.Error(err)
	}
	log.Info(s)
	log.Info("------------------------------------")
}
```

## 启动一个子进程

```go
func (c *Cmd) Start() error

func (c *Cmd) Run() error
func (c *Cmd) CombinedOutput() ([]byte, error)
func (c *Cmd) Output() ([]byte, error)
```

**示例**

```go
package main

import (
	"io/ioutil"
	"os/exec"
	"path/filepath"

	log "github.com/sirupsen/logrus"
)

func main() {
	s, err := exec.LookPath("ifconfig")
	if err != nil {
		log.Fatal("Look Up Command ERROR : ", err)
	}

	folder, filename := filepath.Split(s)

	cmd := exec.Cmd{
		Path: filename,
		Dir:  folder,
	}
	reader, err := cmd.StdoutPipe()
	if err != nil {
		log.Fatal("Stdout Pipe ERROR : ", err)
	}

	if err := cmd.Start(); err != nil {
		log.Fatal("Start ERROR : ", err)
	}

	blist, err := ioutil.ReadAll(reader)
	if err != nil {
		log.Fatal("Read Result ERROR : ", err)
	}
	log.Info(string(blist))
	err = cmd.Wait()
	if err != nil {
		log.Fatal("Wait ERROR : ", err)
	}
	log.Info("------------------------------------")
}
```

> 因为`ifconfig`在部分操作系统（比如debian）做为root命令来使用，所以，需要使用`sudo ./xxxx`来执行编译后的程序。

