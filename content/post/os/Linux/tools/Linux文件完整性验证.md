---
title: "Linux文件完整性验证"
date: 2019-10-14T18:16:13+08:00
categories:
- linux
tags:
- linux
keywords:
- linux
---

在Linux环境下载文件，往往会校验文件的完整性和安全性，即校验文件的hash。常用的hash算法有`MD5`，`SHA1`，`SHA256`，`SHA512`。Linux操作系统操作了方便的校验程序。

<!--more-->

以下以sha256校验为例，展示生成sha256校验文件及文件完整性校验的过程。

## 生成sha256校验信息

```text
$ sha256sum file.name 
```

示例

```text
$ sha256sum go1.13.1.linux-amd64.tar.gz 
94f874037b82ea5353f4061e543681a0e79657f787437974214629af8407d124  go1.13.1.linux-amd64.tar.gz
```

也可以将校验信息输出到文件

```text
$ sha256sum go1.13.1.linux-amd64.tar.gz > SHA256SUM
```

对目录下的所有文件生成校验信息

```text
$ ls | grep -v "SHA256SUM" | xargs sha256sum >> SHA256SUM
$ cat SHA256SUM 
94f874037b82ea5353f4061e543681a0e79657f787437974214629af8407d124  go1.13.1.linux-amd64.tar.gz
68a2297eb099d1a76097905a2ce334e3155004ec08cdea85f24527be3c48e856  go1.13.linux-amd64.tar.gz
```

## 验证文件完整性

```text
$ sha256sum -c --ignore-missing SHA256SUM 
go1.13.1.linux-amd64.tar.gz: 成功
go1.13.linux-amd64.tar.gz: 成功

```

> `-c`选项表示从文件中读取SHA256的校验值并予以检查。`--ignore-missing`表示忽略不存在的文件。
