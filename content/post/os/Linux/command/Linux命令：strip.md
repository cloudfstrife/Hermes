---
title: "Linux命令：strip"
date: 2023-06-14T17:25:45+08:00
lastmod: 2023-06-14T17:25:45+08:00

categories:
- linux
- command
tags:
- linux
- command
- strip
keywords:
- linux
- command
- strip
 - 

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

`strip` 用于剥掉特定文件中的符号信息和调试信息以减小静态库、动态库和程序的大小

<!--more-->

`strip` 支持的选项可通过如下命令查看：

```console
strip --help
```

## strip

```c
// main.c
#include <stdio.h>

char *version = "1.0.0";

int showVersion()
{
    printf("%s\n", version);
}

int main()
{
    return showVersion();
}
```

编译 

```console
gcc -o main main.c
```

查看文件信息

```console
$ file main
main: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a07818aba1988dbb4870289887fbe0f6ba59d23a, for GNU/Linux 3.2.0, not stripped
```
查看符号表

```console
$ nm main
000000000000037c r __abi_tag
0000000000004020 B __bss_start
0000000000004020 b completed.0
                 w __cxa_finalize@GLIBC_2.2.5
0000000000004008 D __data_start
0000000000004008 W data_start
0000000000001080 t deregister_tm_clones
00000000000010f0 t __do_global_dtors_aux
0000000000003dd8 d __do_global_dtors_aux_fini_array_entry
0000000000004010 D __dso_handle
0000000000003de0 d _DYNAMIC
0000000000004020 D _edata
0000000000004028 B _end
0000000000001160 T _fini
0000000000001130 t frame_dummy
0000000000003dd0 d __frame_dummy_init_array_entry
0000000000002108 r __FRAME_END__
0000000000003fe8 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
000000000000200c r __GNU_EH_FRAME_HDR
0000000000001000 T _init
0000000000002000 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 U __libc_start_main@GLIBC_2.34
000000000000114f T main
                 U puts@GLIBC_2.2.5
00000000000010b0 t register_tm_clones
0000000000001139 T showVersion
0000000000001050 T _start
0000000000004020 D __TMC_END__
0000000000004018 D version
```

删除符号表

```console
$ strip main
$ nm main
nm: main：无符号
```

## strip 的使用场景

`strip` 一个目标文件后，其中的符号信息会丢失，但不影响程序的运行，这可以使目标文件变小，在嵌入式开发中，这非常有用。但是因为 `strip` 之后的目标文件丢失了符号信息和调试信息，不方便定位 Bug ，所以在实际的开发中，通常的做法是： strip前的目标文件用于调试， strip后的库用来实际发布