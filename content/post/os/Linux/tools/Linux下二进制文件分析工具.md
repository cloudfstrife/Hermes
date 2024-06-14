---
title: "Linux下二进制文件分析工具"
date: 2024-06-14T16:06:18+08:00
lastmod: 2024-06-14T16:06:18+08:00

categories:
  - Linux
tags:
  - Linux
keywords: 
  - Linux

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Linux 系统提供了很多工具用于分析二进制文件信息，这篇文章将介绍其中一些最流行的 Linux 工具和命令，其中大部分都是 Linux 发行版的一部分。

<!--more-->

## file

作用：帮助确定文件类型

```bash
$ file ./main
./main: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=eb202b6ac03af3a38ce6924d829c1b2d29e09618, for GNU/Linux 3.2.0, not stripped
```
## size

作用： 显示可显示二进制文件中节的大小

```bash
$ size ./main
   text    data     bss     dec     hex filename
   1452     584       8    2044     7fc ./main
```

## ldd

作用：确定可执行程序依赖的共享对象文件

```bash
$ ldd ./main
        linux-vdso.so.1 (0x00007ffd49e8f000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f635a65b000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f635a85a000)
```
## hexdump

作用：以 ASCII、十进制、十六进制或八进制显示文件内容

```bash
$ hexdump -C ./main | head -n 5
00000000  7f 45 4c 46 02 01 01 00  00 00 00 00 00 00 00 00  |.ELF............|
00000010  03 00 3e 00 01 00 00 00  50 10 00 00 00 00 00 00  |..>.....P.......|
00000020  40 00 00 00 00 00 00 00  b0 36 00 00 00 00 00 00  |@........6......|
00000030  00 00 00 00 40 00 38 00  0d 00 40 00 1f 00 1e 00  |....@.8...@.....|
00000040  06 00 00 00 04 00 00 00  40 00 00 00 00 00 00 00  |........@.......|
```

## nm

作用：列出可执行程序中的符号

```bash
$ nm ./main
000000000000037c r __abi_tag
0000000000001139 T add
0000000000004018 B __bss_start
......
```

## ltrace

作用：跟踪库的调用

ltrace 会显示运行时从库中调用的所有函数。包括 函数名称，传递给该函数的参数，函数返回的内容

```bash
ltrace ./main 
printf("%d + %d = %d , c = %d\n", 3, 4, 7, 53 + 4 = 7 , c = 5
)                                                                                                     = 18
+++ exited (status 0) +++
```

> 注意：这里`printf`函数调用的输出内容也显示在这里里

## strace

作用：跟踪系统调用和信号

```bash
$ strace ./main 
execve("./main", ["./main"], 0x7ffff987bcc0 /* 74 vars */) = 0
......
```

## strings

作用：打印文件中的可打印字符的字符串

```bash
$ strings ./main
/lib64/ld-linux-x86-64.so.2
__libc_start_main
......
```

## readelf

作用：显示有关 ELF 文件的信息

```bash
$ readelf -h ./main
ELF 头：  Magic：  7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  类别:                              ELF64
  数据:                              2 补码，小端序 (little endian)
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI 版本:                          0
  类型:                              DYN (Position-Independent Executable file)
  系统架构:                          Advanced Micro Devices X86-64
  版本:                              0x1
  入口点地址：              0x1050
  程序头起点：              64 (bytes into file)
  Start of section headers:          14000 (bytes into file)
  标志：             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

## objdump

作用：还原汇编指令，读取二进制中特定段的信息，还原源代码信息

```bash
$ objdump -d ./main | head -n 10

./main：     文件格式 elf64-x86-64


Disassembly of section .init:

0000000000001000 <_init>:
    1000:       48 83 ec 08             sub    $0x8,%rsp
    1004:       48 8b 05 c5 2f 00 00    mov    0x2fc5(%rip),%rax        # 3fd0 <__gmon_start__@Base>
    100b:       48 85 c0                test   %rax,%rax
```
