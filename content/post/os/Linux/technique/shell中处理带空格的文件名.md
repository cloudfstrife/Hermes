---
title: "shell中处理带空格的文件名"
date: 2019-09-10T18:36:08+08:00
categories:
- Linux
tags:
- Linux
- shell
keywords:
- Linux
- shell
---

在文件系统中，许多文件的文件名带有空格；在Linux系统中默认以空格做为值与值之间的分隔符，shell处理这些文件名时，需要使用特别的处理方式。

<!--more-->

一个示例

```bash
#!/bin/bash

for element in `ls $1`;do  
	full_path="$1/$element"	
	echo  $full_path
done
```

执行

```text
$ ls -alh ~/test/
总用量 8.0K
drwxr-xr-x 2 zdxf zdxf 4.0K 9月  10 15:48  .
drwxr-xr-x 5 zdxf zdxf 4.0K 9月  10 15:47  ..
-rw-r--r-- 1 zdxf zdxf    0 9月  10 15:48 'A Introduction.doc'
-rw-r--r-- 1 zdxf zdxf    0 9月  10 15:48 'Hello World.txt'
$ ./test.sh  ~/test
/home/zdxf/test/A
/home/zdxf/test/Introduction.doc
/home/zdxf/test/Hello
/home/zdxf/test/World.txt

```

## IFS

`IFS`（the Internal Field Separator）内部域分隔符。 原解释如下 

> The Internal Field Separator that is used for word splitting after expansion and to split lines into words with the read built-in command. The default value is “<space><tab><new-line>”.

当Shell处理"命令替换"和"参数替换"时，根据`IFS`的值，默认是 space, tab, newline 来拆解读入的变量，然后对特殊字符进行处理，最后重新组合赋值给该变量。

修正

```bash
#!/bin/bash

OLD_IFS=$IFS
IFS=$(echo -en "\n\b")

for element in `ls $1`;do  
	full_path="$1/$element"	
	echo  $full_path
done

IFS=$OLD_IFS
```

执行

```text
$ ./test.sh ~/test/
/home/zdxf/test//A Introduction.doc
/home/zdxf/test//Hello World.txt

```
