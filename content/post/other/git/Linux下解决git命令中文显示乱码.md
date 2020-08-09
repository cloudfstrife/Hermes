---
title: "Linux下解决git命令中文显示乱码"
date: 2020-08-09T10:21:10+08:00
categories:
- git
tags:
- git
- 中文乱码
keywords:
- git
- 中文乱码
---

在 Linux 环境使用`git`操作文件的时候，如果文件名是中文，会显示形如 `\232\350\346\200......` 的乱码。

解决方案很简单：

```text
$ git config --global core.quotepath false
```

`core.quotepath` 设为 `false` 的话，就不会对 `0x80` 以上的字符进行 `quote` 。
