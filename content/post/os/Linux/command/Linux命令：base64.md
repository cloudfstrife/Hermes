---
title: "Linux命令：base64"
date: 2020-01-31T22:45:13+08:00
categories:
- linux
- command
tags:
- linux
- command
- base64
keywords:
- linux
- command
- base64
---

`base64` 命令用于对文件或者标准输入进行编码和解码。

<!--more-->

## 用法


```text
$ base64 --help
用法：base64 [选项]... [文件]
使用 Base64 编码/解码文件或标准输入输出。

如果没有指定文件，或者文件为"-"，则从标准输入读取。

必选参数对长短选项同时适用。
  -d, --decode          解码数据
  -i, --ignore-garbag   解码时忽略非字母字符
  -w, --wrap=字符数     在指定的字符数后自动换行(默认为76)，0 为禁用自动换行

      --help            显示此帮助信息并退出
      --version         显示版本信息并退出

数据以 RFC 4648 规定的 base64 字母格式进行编码。
解码时，输入数据（编码流）可能包含一些非有效 base64 字符的换行符。
可以尝试用 --ignore-garbage 选项来绕过编码流中的无效字符。

GNU coreutils 在线帮助：<https://www.gnu.org/software/coreutils/>
请向 <http://translationproject.org/team/zh_CN.html> 报告 base64 的翻译错误
完整文档请见：<https://www.gnu.org/software/coreutils/base64>
或者在本地使用：info '(coreutils) base64 invocation'
```

## 示例

编码标准输入


```text
$ base64 
你好
5L2g5aW9Cg==
```

> 在终端输入 `base64` ，执行后，在终端中输入要编码的内容，按 `ctrl+D` 结束输入

编码文件

```text
$ touch testing.txt
$ echo "你好"> testing.txt
$ base64 testing.txt 
5L2g5aW9Cg==
```

解码标准输入

```text
$ base64 -d
5L2g5aW9Cg==
你好
```

解码文件

```text
$ base64 testing.txt  > encoded.txt
$ cat encoded.txt 
5L2g5aW9Cg==
$ base64 -d encoded.txt > decoded.txt
$ cat decoded.txt 
你好
```
