---
title: "Shell命令实现代码行数统计"
date: 2019-10-28T18:10:41+08:00
categories:
- linux
- shell
tags:
- linux
keywords:
- linux
---

Shell命令实现代码行数统计

<!--more-->

```text
find ./ -regextype posix-extended -regex ".*.(md|go)" -type f | xargs -I {} grep -v "^$" {} | wc -l
```

## 解释

```text
find ./ -regextype posix-extended -regex ".*.(md|go)" -type f
```

在当前目录下查找以`.conf`,`.md`,`.go`为后缀的文件。

---

```text
xargs -I {} grep -v "^$" {}
```

查看这些文件的内容，并去除空行

---

```text
wc -l
```

计算行数

## 可能遇到的问题

* 空格问题

如果代码文件名中有空格(一般貌似不会这样，但是其它类型的文件可能会存在这样的情况)，需要特别处理。参见：[Shell中处理带空格的文件名 ](/post/os/linux/technique/shell中处理带空格的文件名)

