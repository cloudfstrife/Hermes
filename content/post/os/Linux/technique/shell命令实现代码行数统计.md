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

如果代码文件名中有空格(一般貌似不会这样，但是其它类型的文件可能会存在这样的情况)，需要特别处理。参见：[Shell中处理带空格的文件名 ](https://www.bitlogs.tech/2019/09/shell%E4%B8%AD%E5%A4%84%E7%90%86%E5%B8%A6%E7%A9%BA%E6%A0%BC%E7%9A%84%E6%96%87%E4%BB%B6%E5%90%8D/)

