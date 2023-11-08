---
title: "Go子进程"
date: 2019-07-30T18:05:44+08:00
categories:
- Go
tags:
- Go
keywords:
- Go
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
// 开始执行命令，但是并不等待程序执行结束
func (c *Cmd) Start() error

// 开始执行命令，并等待程序执行结束
func (c *Cmd) Run() error

// 开始执行命令，并等待程序执行结束，如果程序有异常或者输出，则合并返回
func (c *Cmd) CombinedOutput() ([]byte, error)

// 开始执行命令，并等待程序执行结束，如果程序有输出，则返回
func (c *Cmd) Output() ([]byte, error)
```

**Start 示例**

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

**Run 示例**

```go
package main

import (
	"os/exec"
	"path/filepath"

	log "github.com/sirupsen/logrus"
)

func init() {
	log.SetFormatter(&log.TextFormatter{
		FullTimestamp: true,
	})
}

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
	if err := cmd.Run(); err != nil {
		log.Fatal("Wait ERROR : ", err)
	}
	log.Info("------------------------------------")
}

```

**Output 示例**

```go
package main

import (
	"os/exec"
	"path/filepath"

	log "github.com/sirupsen/logrus"
)

func init() {
	log.SetFormatter(&log.TextFormatter{
		FullTimestamp: true,
	})
}

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
	var blist []byte
	if blist, err = cmd.Output(); err != nil {
		log.Fatal("Wait ERROR : ", err)
	}
	log.Info("Result :\n", string(blist))
	log.Info("------------------------------------")
}

```

> 因为`ifconfig`在部分操作系统（比如debian）做为root命令来使用，所以，需要使用`sudo ./xxxx`来执行编译后的程序。

## 结束进程

```go
cmd.Process.Kill()
cmd.Wait()
```

在结束子进程时，父进程还没有释放资源，所以子进程不能完成资源清理工作。此时可以使用`cmd.Wait()`完成子进程资源清理工作。

**示例**

```go
package main

import (
	"io/ioutil"
	"os/exec"
	"path/filepath"
	"time"

	log "github.com/sirupsen/logrus"
)

func init() {
	log.SetFormatter(&log.TextFormatter{
		FullTimestamp: true,
	})
}

func main() {
	s, err := exec.LookPath("top")
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
	log.Info("top run with pid : ", cmd.Process.Pid)

	blist, err := ioutil.ReadAll(reader)
	if err != nil {
		log.Fatal("Read Result ERROR : ", err)
	}
	log.Info(string(blist))

	time.Sleep(5 * time.Second)

	err = cmd.Process.Kill()
	if err != nil {
		log.Fatal("Kill Sub Process ERROR : ", err)
	}

	log.Info("Killed")

	time.Sleep(30 * time.Second)

	err = cmd.Wait()
	if err != nil {
		log.Fatal("Wait ERROR : ", err)
	}
	log.Info("------------------------------------")
}

```

> 上面的示例中，当终端输出`Killed`时，在另一个终端执行`ps -ef | grep sub_pid`，会得到如下输出：

```text
$ ps -ef | grep 3647
USER      3647   3641  0 18:53 pts/0    00:00:00 [top] <defunct>
USER      3649   3636  0 18:54 pts/1    00:00:00 grep 3647
```

