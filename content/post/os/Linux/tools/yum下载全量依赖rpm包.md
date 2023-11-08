---
title: "yum下载全量依赖rpm包"
date: 2022-08-19T10:21:11+08:00
categories:
- Linux
tags:
- linux
- yum
keywords:
- linux
- yum
#thumbnailImage: //example.com/image.jpg
---

对于一些安全级别比较高的生产环境，服务器一般都无法访问互联网，如果需要安装软件，就需要进行离线安装。

CentOS离线安装的方法主要有两种：源码编译、rpm安装包。通常采用 rpm 包安装。

<!--more-->

## 查看依赖

```text
$ sudo yum deplist xxxx
```

## 下载

### 使用 repotrack

安装

```text
$ sudo yum install yum-utils
```

下载

```text
$ repotrack xxxx
```

### 使用 yumdownloader

安装

```text
$ sudo yum install yum-utils
```

下载

```text
$ yumdownloader --resolve --destdir=. xxxx
```

### 使用 yum 的 downloadonly 插件


安装

```text
$ sudo yum install yum-download
```

下载

```text
$ yum -y install xxxx --downloadonly --downloaddir=.
```

## 安装

```text
$ rpm -Uvh --force --nodeps *.rpm
```