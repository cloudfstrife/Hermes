---
title: "Windows查找占用指定端口的进程"
date: 2019-12-27T16:19:30+08:00
categories:
- other
- windows
tags:
- windows
- port
keywords:
- windows
- port
---

在windows环境下，当指定端口被占用时，可以使用命令行来查找被哪个进程占用，进一步杀死进程

<!--more-->

## 查找占用指定端口的进程

```text
netstat -aon | findstr %PORT_NUMBER%
```

> `%PORT_NUMBER%` 为指定端口号

## 杀死进程

```text
taskkill /pid %PID% -t -f 
```

> `%PID%` 为指定进程号

## 示例

```text
$ netstat -aon | findstr 8081
  TCP    0.0.0.0:8081           0.0.0.0:0              LISTENING       14412
  TCP    [::]:8081              [::]:0                 LISTENING       14412
$ taskkill /pid 14412 -t -f 
成功: 已终止 PID 14412 (属于 PID 2692 子进程)的进程。

```
