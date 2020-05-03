---
title: "Linux查找最大的N个文件"
date: 2020-04-20T19:17:18+08:00
categories:
- linux
- shell
tags:
- linux
- 最大文件
keywords:
- linux
- 最大文件
---

当系统的磁盘空间不足时，需要查找目录下最大的文件，用于释放磁盘空间。

<!--more-->

## 命令

```text
find . -type f -print0 | xargs -0 du -m | sort -rh | head -n N
```

### 解释

> `find . -type f -print0` : 搜索当前目录( `.` )下的所有文件( `-type f` )，并在标准输出显示完整的文件名，每个文件后面跟一个空字符( `NULL` ) ( `-print0` )
>
> `xargs -0 du -m` : 从标准输入读取( `xargs` )数据做为参数，以 `NULL` 分割( `-0` )，并计算每个输入的磁盘占用情况( `du` )，以MB为单位输出( `-m` )
>
> `sort -rh` : 对文本文件进行排序( `sort` )，排序为逆向( `-r` ) ，以人类格式输出( `-h` )
>
> `head -n N` : 输出开头部分( `head` )，输出前N个( `-n N` )
