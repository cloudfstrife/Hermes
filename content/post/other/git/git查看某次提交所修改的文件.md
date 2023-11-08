---
title: "Git查看某次提交所修改的文件"
date: 2020-08-09T10:11:43+08:00
categories:
- git
tags:
- git
keywords:
- git
---

有时候想快速查看某个提交中，修改了哪些文件，操作可以分为： 

1. 通过 `git log` 获取提交的 commit ID
2. 使用 `git show` 查看修改内容

<!--more-->

## 通过 `git log` 获取提交的 commit ID

查看所有提交

```text
$ git log 
```

查看指定文件的提交 

```text
$ git log filename
```

查看修改了哪些文件

```text
$ git log --stat
```

## 使用 `git show` 查看修改内容

```text
$ git show Commit ID
```

