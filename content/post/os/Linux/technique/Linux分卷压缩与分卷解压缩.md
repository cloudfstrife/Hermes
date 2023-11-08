---
title: "Linux分卷压缩与分卷解压缩"
date: 2022-07-28T18:59:29+08:00
categories:
- linux
- tar
tags:
- linux
- tar
- split
keywords:
- linux
- tar
- split
---

Linux分卷压缩与分卷解压缩，这里采用常用的是 `tar` 命令与 `split` 命令结合完成会卷操作。

<!--more-->

## 压缩

```text
$ tar zcvf - ${FILEPATH} |split -b ${SIZE} -d - ${package_prefix_name}
```

示例

```text
tar zcvf - Hermes |split -b 1m -d - hermes.tar.gz
```

## 解压缩

```text
$ cat ${package_prefix_name}* | tar zxvf 
```

示例

```text
$ cat hermes.tar.gz0* | tar zxvf -
```