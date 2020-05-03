---
title: "Linux命令：time"
date: 2020-02-25T12:47:13+08:00
categories:
- linux
- command
tags:
- linux
- command
- time
keywords:
- linux
- command
- time
---

`time` 命令主要用于测量命令执行时所消耗的时间及系统资源信息。

<!--more-->

`time` 命令执行命令，在命令执行结束时，显示命令执行过程中使用的资源信息（默认输出到标准错误输出），如果命令异常退出，则显示警告消息和退出状态。

## 安装 

>  bash 内置了 `time` 命令，执行 `time` 命令时，可以以绝对路径执行或者前面加 `\` 来运行

```text
$ time ls
Downloads

real	0m0.003s
user	0m0.002s
sys	0m0.001s
$ \time ls
Downloads
0.00user 0.00system 0:00.00elapsed 100%CPU (0avgtext+0avgdata 2292maxresident)k
0inputs+0outputs (0major+114minor)pagefaults 0swaps
```

Debian

```text
$ sudo apt-get install time
```

## 常用选项

* `-o` FILE 或者 `--output`=FILE

将资源使用统计信息写入FILE而不是标准错误流。

* `-a` `--append`

将资源使用信息附加到输出文件，而不是覆盖它。

* `-f` FORMAT `--format` FORMAT

指定 `time` 输出的格式字符串

* `-p` `--portability`

使用以下格式字符串，以符合POSIX 1003.2标准：

```text
real %e
user %U
sys %S
```

* `-v` `--verbose`

使用内置的详细格式，该格式在单独的行上显示程序资源使用情况的每条可用信息，并以英文形式对其进行描述

* `--quiet`

即使命令异常退出，也不需要报告程序的状态

* `-V` `--version`

输出版本信息

## 示例

版本信息

```text
$ /usr/bin/time --version
GNU time 1.7
```

一般输出

```text
$ /usr/bin/time wc nohup.out 
  18283  207266 2713917 nohup.out
0.05user 0.00system 0:01.23elapsed 4%CPU (0avgtext+0avgdata 1952maxresident)k
5392inputs+0outputs (1major+83minor)pagefaults 0swaps
```

标准POSIX 1003.2输出

```text
$ /usr/bin/time -p wc nohup.out 
  18283  207266 2713917 nohup.out
real 0.05
user 0.04
sys 0.00
```

详细输出

```text
$ /usr/bin/time -v wc nohup.out 
  18283  207266 2713917 nohup.out
	Command being timed: "wc nohup.out"
	User time (seconds): 0.04
	System time (seconds): 0.00
	Percent of CPU this job got: 96%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:00.05
	Average shared text size (kbytes): 0
	Average unshared data size (kbytes): 0
	Average stack size (kbytes): 0
	Average total size (kbytes): 0
	Maximum resident set size (kbytes): 2004
	Average resident set size (kbytes): 0
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 85
	Voluntary context switches: 0
	Involuntary context switches: 84
	Swaps: 0
	File system inputs: 0
	File system outputs: 0
	Socket messages sent: 0
	Socket messages received: 0
	Signals delivered: 0
	Page size (bytes): 4096
	Exit status: 0
```

## 格式化输出

格式化输出`time` 命令的输出信息可以由 `--format` 指定，如果命令没有指定这个参数，则由使用 `TIME` 环境变量做为格式化信息，如果 `TIME` 环境变量没有指定，则使用默认格式。默认格式为

```text
%Uuser %Ssystem %Eelapsed %PCPU (%Xtext+%Ddata %Mmax)k
%Iinputs+%Ooutputs (%Fmajor+%Rminor)pagefaults %Wswaps
```

格式字符串通常由纯文本中的 **资源说明符** 组成，以百分号开头的以下词将被解析为资源说明符，`\t` 输出制表符，`\n` 输出换行符，`\\` 输出反斜杠。格式字符串中的其他文本将原样复制到输出中。`time` 会在输出结束时，增加一个新行，所以格式化字符不需要以 `\n` 结尾。

| 标识 | 含义                                                                             |
| ---- | -------------------------------------------------------------------------------- |
| %    | `%`                                                                              |
| C    | time启动的命令名和参数                                                           |
| D    | 进程的非共享数据区域的平均大小，以千字节为单位。                                 |
| E    | 执行命令所花费的时间，格式是：[hour]:minute:second                               |
| F    | 进程运行时的主要内存页错误的数量                                                 |
| I    | 进程运行时文件系统输入的数量                                                     |
| K    | 进程的平均总内存使用量，以千字节为单位                                           |
| M    | 执行程序所占用内存的最大值，以千字节为单位                                       |
| O    | 进程运行时文件系统输出的数量                                                     |
| P    | 执行指令时 CPU 的占用比例，为用户+系统时间除以总运行时间，它还会打印一个百分号。 |
| R    | 此程序的次要内存页错误发生的次数                                                 |
| S    | 进程使用的CPU总秒数（在内核模式下），以秒为单位。                                |
| U    | 进程使用的CPU总秒数（在用户模式下），以秒为单位。                                |
| W    | 进程从主内存中换出的次数。                                                       |
| X    | 执行命令间共享内容数量，以千字节为单位。                                         |
| Z    | 系统的页面大小，以字节为单位。                                                   |
| c    | 此程序被强制中断（分配到的 CPU 时间耗尽）的次数                                  |
| e    | 进程使用的实际时间，以秒为单位                                                   |
| k    | 传递到进程的信号数。                                                             |
| p    | 进程的平均非共享堆栈大小，以千字节为单位                                         |
| r    | 此程序所收到的 Socket Message                                                    |
| s    | 此程序所发出的 Socket Message                                                    |
| t    | 执行程序所占用的内存总量（stack+data+text）的平均大小， 单位是 KB                |
| w    | 程序自动进行上下文切换的次数，                                                   |
| x    | 命令的结束代码                                                                   |

Format 示例

```text
$ \time --format="%W swap %P process" ls 
Downloads
0 swap 100% process
$ \time -f "%W swap %P process" ls 
Downloads
0 swap 50% process
```
