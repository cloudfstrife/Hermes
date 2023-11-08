---
title: "Shell截取字符串"
date: 2021-08-11T15:31:42+08:00
categories:
- Linux
- shell
tags:
- Linux
- substring
keywords:
- Linux
- substring
---

Linux shell 中截取字符串的方式有以下几种，可以跟据 shell 的场景自行使用。

<!--more-->

# awk中函数substr

```shell
#!/usr/bin/env bash

v="13579"

A=`echo $v | awk '{print substr($0,2,3)}'`
echo $A
```

> 索引以1开始

# expr substr

```shell 
#!/usr/bin/env bash

v="13579"

A=`expr substr $v 2 3`
echo $A
```

> 索引以1开始

# 使用字符索引

```shell
#!/usr/bin/env bash

v="13579"

A=${v:1:3}
echo $A
```

> 索引以0开始

# 使用\#和％截取

```shell
#!/usr/bin/env bash

v="/tmp/go-build733636567/b093/cgo.cgo1.go"

# 从左向右，切除第一个斜线及前面的内容
echo ${v#\/*}
# 从左向右，切除最后一个斜线及前面的内容
echo ${v##*\/}
# 从右向左，切除第一个斜线及后面的内容
echo ${v%\/*}
# 从右向左，截取最后一个斜线及后面的
echo ${v%%\/*}
```
