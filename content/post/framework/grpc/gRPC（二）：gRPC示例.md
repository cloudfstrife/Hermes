---
title: "gRPC（二）：gRPC示例"
date: 2020-05-04T17:50:27+08:00
categories:
- gRPC
tags:
- gRPC
keywords:
- gRPC
---

本文通过一个示例，简单介绍gRPC开发环境搭建及开发过程

<!--more-->

## 构建gRPC环境

### 操作系统及Go版本

```text
$ cat /etc/debian_version 
10.3
$ go version
go version go1.14.2 linux/amd64
```

### 安装protoc

```text
$ mkdir -p ~/Downloads/tmp
$ wget https://github.com/protocolbuffers/protobuf/releases/download/v3.14.0/protoc-3.14.0-linux-x86_64.zip -O ~/Downloads/tmp/protoc-3.11.4-linux-x86_64.zip
$ mkdir -p ~/Downloads/tmp/protoc
$ unzip ~/Downloads/tmp/protoc-3.14.0-linux-x86_64.zip -d ~/Downloads/tmp/protoc/
$ sudo cp ~/Downloads/tmp/protoc/bin/protoc /usr/local/bin/
```

> 下载文件路径可以跟据最新版本的变化修改 [Protocol Buffers - Google's data interchange format](https://github.com/protocolbuffers/protobuf)

### 安装protobuf Protoc Plugin

```text
$ mkdir -p ~/Downloads/tmp
$ wget https://github.com/protocolbuffers/protobuf-go/releases/download/v1.25.0/protoc-gen-go.v1.25.0.linux.amd64.tar.gz -O ~/Downloads/tmp/protoc-gen-go.v1.25.0.linux.amd64.tar.gz
$ mkdir -p ~/Downloads/tmp/protoc-gen-go
$ tar xf ~/Downloads/protoc-gen-go.v1.21.0.linux.amd64.tar.gz -C ~/Downloads/tmp/protoc-gen-go/
$ sudo cp ~/Downloads/tmp/protoc-gen-go/protoc-gen-go /usr/local/bin/
```

> 下载文件路径可以跟据最新版本的变化修改 [Go support for Protocol Buffers](https://github.com/protocolbuffers/protobuf-go)
>
> 之前的Go语言Protocol Buffers支持 [github.com/golang/protobuf](https://pkg.go.dev/mod/github.com/golang/protobuf) 已经被新的库 [google.golang.org/protobuf](https://pkg.go.dev/mod/google.golang.org/protobuf)

### 安装gRPC Plugin

```text
$ mkdir -p ~/Downloads/tmp
$ cd ~/Downloads/tmp
$ git clone https://github.com/grpc/grpc-go.git
$ cd grpc-go/cmd/protoc-gen-go-grpc 
$ go build 
$ sudo cp protoc-gen-go-grpc /usr/local/bin/
```

> 官方站点的示例里，使用的`go get google.golang.org/protobuf/cmd/protoc-gen-go google.golang.org/grpc/cmd/protoc-gen-go-grpc`来完成插件安装的
> 
> 但是国内因为GFW的问题，只能手工完成

## 代码

### 初始化结构

```text
$ mkdir -p grpc-test/cmd/{server,client} grpc-test/proto
$ touch grpc-test/cmd/{server,client}/main.go  grpc-test/proto/echo.proto
$ cd grpc-test/
$ go mod init app/grpc-test
```

**proto/echo.proto**

```proto3
syntax="proto3";

package echo;

option go_package="rpc/echo";

message EchoPar {
    string msg = 1;
}

message EchoReply {
    string msg = 1;
}

service echo {
    rpc Echo(EchoPar) returns ( EchoReply) {}
}
```

> 这里定义了两个 message 和一个 RPC 服务

### 生成gRPC代码

```text
$ protoc --go_out=. --go-grpc_out=. proto/echo.proto
```

### 实现服务端

**cmd/server/main.go**

```go
package main

import (
	"app/grpc-test/rpc/echo"
	"context"
	"net"
	"log"

	"google.golang.org/grpc"
)

type EchoServer struct {
	echo.UnimplementedEchoServer
}

func (e *EchoServer) Echo(ctx context.Context, par *echo.EchoPar) (*echo.EchoReply, error) {
	result := &echo.EchoReply{
		Msg: par.GetMsg(),
	}
	return result, nil
}

func main() {
	log.Println("Go")
	listen, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	echo.RegisterEchoServer(s, &EchoServer{})
	if err := s.Serve(listen); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

> `EchoServer` 实现了 `Echo` 方法，执行具体的业务
> 
> `echo.RegisterEchoServer(s, &EchoServer{})` 将实现类注册到 grpc Server

### 客户端代码

**cmd/client/main.go**

```go
package main

import (
	"app/grpc-test/rpc/echo"
	"context"
	"log"

	"google.golang.org/grpc"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:8080", grpc.WithInsecure(), grpc.WithBlock())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := echo.NewEchoClient(conn)
	ctx, cancle := context.WithCancel(context.Background())
	defer cancle()
	r, err := c.Echo(ctx, &echo.EchoPar{Msg: "hello"})
	if err != nil {
		log.Fatalf("echo failed :%#v", err)
	}
	log.Print(r.GetMsg())
}
```

### 编译与运行

```
$ go build ./cmd/server
$ ./server
```

```
$ go build ./cmd/client 
$ ./client
```
## 总结

使用Go语言实现 gRPC 服务的过程如下：

### 服务端

* 安装环境
* 编写 proto 文件
* 生成 Go 文件
* 实现业务方法
* 创建gRPC server （ `grpc.NewServer()` ）
* 调用生成的注册方法( `RegisterXXXXXX` )注册实现
* 调用 `Serve` 方法启动服务

### 客户端

* 创建连接 `grpc.Dial`
* 创建 Client 
* 调用实际的 gRPC 方法