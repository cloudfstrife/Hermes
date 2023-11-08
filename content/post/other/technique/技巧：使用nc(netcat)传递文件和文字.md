---
title: "技巧：使用nc(netcat)传递文件和文字"
date: 2022-10-22T11:58:32+08:00
categories:
- other
- technique
tags:
- technique
- nc
keywords:
- technique
- nc
---

使用 netcat 传递文件和文字

<!--more-->

## 批量发送文件

> 本文使用 `netcat-openbsd` 版本的 `netcat`
>
> 监听端口以 `10000` 为例，发送 `~/Downloads/` 目录下的所有文件

接收端

```text
$ nc -l -p 10000 | tar zxvf - -C ~/Downloads/
```

发送端

```text
$ tar zcf - ~/Downloads/* | nc xxx.xxx.x.xx 10000 -q 1
```

## 发送文本

接收端

```text
$ nc -l -p 10000
```

发送端

```text
$ cat <<EOF | nc xxx.xxx.x.xx 10000 -q 1
此处输入文本
EOF
```
