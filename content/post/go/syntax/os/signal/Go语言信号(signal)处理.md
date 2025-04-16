---
title: "Go语言信号(signal)处理"
date: 2019-11-04T18:31:30+08:00
categories:
- go
tags:
- go
- signal
keywords:
- go
- signal
---

在某些应用中（尤其是生产级程序），需要处理一些系统信号(signal)。比如，当程序收到`SIGINT`信号时，清理资源，优雅的退出程序。  
Go语言的`os/signal`包提供了系统信号监听机制，用于实现系统信号处理。

<!--more-->

关于Linux信号，参见[Linux信号机制](/post/os/linux/system/linux信号机制)

## os/signal包的函数

```go
func Ignore(sig ...os.Signal)
func Ignored(sig os.Signal) bool
func Notify(c chan<- os.Signal, sig ...os.Signal)
func Reset(sig ...os.Signal)
func Stop(c chan<- os.Signal)
```

**func Ignore(sig ...os.Signal)**

`Ignore` 用于忽略指定的信号，如果没有输入参数，则所有信号都将被忽略。

**func Ignored(sig os.Signal) bool**

`Ignored` 返回指定信号当前是否处于被忽略状态。

**func Notify(c chan<- os.Signal, sig ...os.Signal)**

`Notify` 用于将系统信号分发到`c chan `。  
如果没有信号参数输入，则所有信号都将发送到`c`。发送动作是无阻塞的，所以要求c有足够的缓冲区空间，以跟得上系统的信号速率。  
此方法允许 在同一个通道上多次调用`Notify`，也允许将同一种信号分发到不同的`chan`，每个`chan`独立接收输入信号的副本。

**func Reset(sig ...os.Signal)**

`Reset`将取消先前调用`Notify`的效果。如果没有信号参数输入，则所有信号都将取消。

**func Stop(c chan<- os.Signal)**

`Stop`操作将停止向`c chan`发送信号。此方法用于取消所有之前对此`chan`调用`Notify`的效果。

## 项目示例

[https://github.com/etcd-io/etcd/blob/master/pkg/osutil/interrupt_unix.go](https://github.com/etcd-io/etcd/blob/master/pkg/osutil/interrupt_unix.go)

在`ectd`的源码中，`pkg/osutil/interrupt_unix.go`的`HandleInterrupts`方法，监听系统`SIGINT`与`SIGTERM`信号，并在收到信号之后，执行清理工作。