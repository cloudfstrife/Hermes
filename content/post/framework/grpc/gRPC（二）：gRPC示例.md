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
$ mkdir -p ~/Downloads
$ wget https://github.com/protocolbuffers/protobuf/releases/download/v3.11.4/protoc-3.11.4-linux-x86_64.zip -O ~/Downloads/protoc-3.11.4-linux-x86_64.zip
$ mkdir -p ~/Downloads/protoc
$ unzip ~/Downloads/protoc-3.11.4-linux-x86_64.zip -d ~/Downloads/protoc/
$ sudo cp ~/Downloads/protoc/bin/protoc /usr/local/bin/
```

> 下载文件路径可以跟据最新版本的变化修改 [Protocol Buffers - Google's data interchange format](https://github.com/protocolbuffers/protobuf)

### 安装protobuf Protoc Plugin


```text
$ mkdir -p ~/Downloads
$ wget https://github.com/protocolbuffers/protobuf-go/releases/download/v1.21.0/protoc-gen-go.v1.21.0.linux.amd64.tar.gz -O ~/Downloads/protoc-gen-go.v1.21.0.linux.amd64.tar.gz
$ mkdir -p ~/Downloads/protoc-gen-go
$ tar xf ~/Downloads/protoc-gen-go.v1.21.0.linux.amd64.tar.gz -C ~/Downloads/protoc-gen-go/
$ sudo cp ~/Downloads/protoc-gen-go/protoc-gen-go /usr/local/bin/
```

> 下载文件路径可以跟据最新版本的变化修改 [Go support for Protocol Buffers](https://github.com/protocolbuffers/protobuf-go)
>
> 之前的Go语言Protocol Buffers支持 [github.com/golang/protobuf](https://pkg.go.dev/mod/github.com/golang/protobuf) 已经被新的库 [google.golang.org/protobuf](https://pkg.go.dev/mod/google.golang.org/protobuf)

### 测试

```text
$ protoc --version 
libprotoc 3.11.4
```

## 示例代码

```text
$ mkdir -p grpc-test/
$ cd grpc-test/
$ go mod init app/grpc-test
$ mkdir -p cmd/server/ cmd/client
$ touch cmd/server/server.go
$ touch cmd/client/client.go
$ mkdir -p proto/
$ touch proto/hello.proto
```
