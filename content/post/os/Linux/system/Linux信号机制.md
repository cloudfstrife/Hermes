---
title: "Linux信号机制"
date: 2019-10-30T19:31:11+08:00
categories:
- Linux
tags:
- Linux
- signal
keywords:
- Linux
- signal
---

软中断信号(signal，又简称为信号)用来通知进程发生了异步事件。内核可以因为内部事件而给进程发送信号，通知进程发生了某个事件。

<!--more-->

## 信号的产生

信号的产生有两个来源：
* 硬件(特定的输入：`ctrl+c`,`ctrl+\`,`ctrl+z`等，硬件故障等)
* 软件（系统函数：kill, raise, alarm和setitimer以及sigqueue等）。

## 标准信号

Linux支持下面列出的标准信号。表格的第二栏指出了指定信号的标准:  

> `P1990`表示该信号在原始POSIX.1-1990标准中进行了描述。
> `P2001`表示信号已添加到SUSv2和POSIX.1-2001中。

| Signal    | Standard | Action | Comment                                |
| --------- | -------- | ------ | -------------------------------------- |
| SIGABRT   | P1990    | Core   | 来自abort的异常信号                    |
| SIGALRM   | P1990    | Term   | 来自alarm的计时器到时信号              |
| SIGBUS    | P2001    | Core   | 总线错误（内存访问错误）               |
| SIGCHLD   | P1990    | Ign    | 子进程停止或终止                       |
| SIGCLD    | -        | Ign    | 与 SIGCHLD 同义                        |
| SIGCONT   | P1990    | Cont   | 如果已停止，继续执行                   |
| SIGEMT    | -        | Term   | Emulator trap                          |
| SIGFPE    | P1990    | Core   | 浮点例外                               |
| SIGHUP    | P1990    | Term   | 终端挂起或者控制进程终止               |
| SIGILL    | P1990    | Core   | 非法指令                               |
| SIGINFO   | -        | -      | 与 SIGPWR 同义                         |
| SIGINT    | P1990    | Term   | 来自键盘的中断信号                     |
| SIGIO     | -        | Term   | IO设备就绪 (4.2BSD)                    |
| SIGIOT    | -        | Core   | IOT自陷，与 SIGABRT 同义               |
| SIGKILL   | P1990    | Term   | 杀死                                   |
| SIGLOST   | -        | Term   | 文件锁丢失 (unused)                    |
| SIGPIPE   | P1990    | Term   | 管道损坏：向一个没有读进程的管道写数据 |
| SIGPOLL   | P2001    | Term   | Pollable事件发生(Sys V)，与 SIGIO 同义 |
| SIGPROF   | P2001    | Term   | 分析计时器到时                         |
| SIGPWR    | -        | Term   | 电力故障 (System V)                    |
| SIGQUIT   | P1990    | Core   | 来自键盘的离开信号                     |
| SIGSEGV   | P1990    | Core   | 段非法错误(内存引用无效)               |
| SIGSTKFLT | -        | Term   | 协处理器堆栈错误 (unused)              |
| SIGSTOP   | P1990    | Stop   | 非来自终端的停止信号                   |
| SIGTSTP   | P1990    | Stop   | 来自终端的停止信号                     |
| SIGSYS    | P2001    | Core   | 非法系统调用 (SVr4);                   |
| SIGTERM   | P1990    | Term   | 终止                                   |
| SIGTRAP   | P2001    | Core   | 跟踪/断点自陷                          |
| SIGTTIN   | P1990    | Stop   | 后台进程读终端                         |
| SIGTTOU   | P1990    | Stop   | 后台进程写终端                         |
| SIGUNUSED | -        | Core   | 未使用信号 与 SIGSYS 同义              |
| SIGURG    | P2001    | Ign    | socket紧急信号 (4.2BSD)                |
| SIGUSR1   | P1990    | Term   | 用户自定义信号1                        |
| SIGUSR2   | P1990    | Term   | 用户自定义信号2                        |
| SIGVTALRM | P2001    | Term   | 虚拟计时器到时 (4.2BSD)                |
| SIGXCPU   | P2001    | Core   | 超过CPU时限 (4.2BSD);                  |
| SIGXFSZ   | P2001    | Core   | 超过文件长度限制 (4.2BSD);             |
| SIGWINCH  | -        | Ign    | 窗口大小改变 (4.3BSD, Sun)             |

`SIGKILL`和`SIGSTOP`信号不能被捕获、阻塞或忽略。

### 标准信号

下表列出了每个信号的数值。在不同的架构上，许多信号具有不同的数值。

| Signal    | x86 / ARM / most others | Alpha/SPARC | MIPS | PARISC | Notes         |
| --------- | ----------------------- | ----------- | ---- | ------ | ------------- |
| SIGHUP    | 1                       | 1           | 1    | 1      |               |
| SIGINT    | 2                       | 2           | 2    | 2      |               |
| SIGQUIT   | 3                       | 3           | 3    | 3      |               |
| SIGILL    | 4                       | 4           | 4    | 4      |               |
| SIGTRAP   | 5                       | 5           | 5    | 5      |               |
| SIGABRT   | 6                       | 6           | 6    | 6      |               |
| SIGIOT    | 6                       | 6           | 6    | 6      |               |
| SIGBUS    | 7                       | 10          | 10   | 10     |               |
| SIGEMT    | -                       | 7           | 7    | -      |               |
| SIGFPE    | 8                       | 8           | 8    | 8      |               |
| SIGKILL   | 9                       | 9           | 9    | 9      |               |
| SIGUSR1   | 10                      | 30          | 16   | 16     |               |
| SIGSEGV   | 11                      | 11          | 11   | 11     |               |
| SIGUSR2   | 12                      | 31          | 17   | 17     |               |
| SIGPIPE   | 13                      | 13          | 13   | 13     |               |
| SIGALRM   | 14                      | 14          | 14   | 14     |               |
| SIGTERM   | 15                      | 15          | 15   | 15     |               |
| SIGSTKFLT | 16                      | -           | -    | 7      |               |
| SIGCHLD   | 17                      | 20          | 18   | 18     |               |
| SIGCLD    | -                       | -           | 18   | -      |               |
| SIGCONT   | 18                      | 19          | 25   | 26     |               |
| SIGSTOP   | 19                      | 17          | 23   | 24     |               |
| SIGTSTP   | 20                      | 18          | 24   | 25     |               |
| SIGTTIN   | 21                      | 21          | 26   | 27     |               |
| SIGTTOU   | 22                      | 22          | 27   | 28     |               |
| SIGURG    | 23                      | 16          | 21   | 29     |               |
| SIGXCPU   | 24                      | 24          | 30   | 12     |               |
| SIGXFSZ   | 25                      | 25          | 31   | 30     |               |
| SIGVTALRM | 26                      | 26          | 28   | 20     |               |
| SIGPROF   | 27                      | 27          | 29   | 21     |               |
| SIGWINCH  | 28                      | 28          | 20   | 23     |               |
| SIGIO     | 29                      | 23          | 22   | 22     |               |
| SIGPOLL   | -                       | -           | -    | -      | Same as SIGIO |
| SIGPWR    | 30                      | 29/-        | 19   | 19     |               |
| SIGINFO   | -                       | 29/-        | -    | -      |               |
| SIGLOST   | -                       | -/29        | -    | -      |               |
| SIGSYS    | 31                      | 12          | 12   | 31     |               |
| SIGUNUSED | 31                      | -           | -    | 31     |               |

## 实时信号

从2.2版开始，Linux支持最初在POSIX.1b实时扩展中定义的实时信号（现在包括在POSIX.1-2001中）。支持的实时信号范围由宏`SIGRTMIN`和`SIGRTMAX`定义。POSIX.1-2001要求实现至少支持`_POSIX_RTSIG_MAX`实时信号。

Linux内核支持33种不同的实时信号，范围从32到64。但是`glibc`POSIX线程实现在内部使用两个（对于NPTL）或三个（对于LinuxThreads）实时信号。并调整`SIGRTMIN`的值(34或35)。实时信号的范围在UNIX系统上会有所不同，因此请不要使用硬编码数字来引用实时信号，而应始终使用`SIGRTMIN + n`表示法来引用实时信号。

与标准信号不同，实时信号没有预定义的含义：所有实时信号都可用于应用程序定义。

## Linux信号值 

在Linux操作系统中，可以使用`kill -l`查看支持信号列表。  
Linux支持POSIX标准信号 和 POSIX实时信号。

```text
$ kill -l
 1) SIGHUP          2) SIGINT           3) SIGQUIT            4) SIGILL             5) SIGTRAP
 6) SIGABRT         7) SIGBUS           8) SIGFPE             9) SIGKILL           10) SIGUSR1
11) SIGSEGV        12) SIGUSR2         13) SIGPIPE           14) SIGALRM           15) SIGTERM
16) SIGSTKFLT      17) SIGCHLD         18) SIGCONT           19) SIGSTOP           20) SIGTSTP
21) SIGTTIN        22) SIGTTOU         23) SIGURG            24) SIGXCPU           25) SIGXFSZ
26) SIGVTALRM      27) SIGPROF         28) SIGWINCH          29) SIGIO             30) SIGPWR
31) SIGSYS         34) SIGRTMIN        35) SIGRTMIN+1        36) SIGRTMIN+2        37) SIGRTMIN+3
38) SIGRTMIN+4     39) SIGRTMIN+5      40) SIGRTMIN+6        41) SIGRTMIN+7        42) SIGRTMIN+8
43) SIGRTMIN+9     44) SIGRTMIN+10     45) SIGRTMIN+11       46) SIGRTMIN+12       47) SIGRTMIN+13
48) SIGRTMIN+14    49) SIGRTMIN+15     50) SIGRTMAX-14       51) SIGRTMAX-13       52) SIGRTMAX-12
53) SIGRTMAX-11    54) SIGRTMAX-10     55) SIGRTMAX-9        56) SIGRTMAX-8        57) SIGRTMAX-7
58) SIGRTMAX-6     59) SIGRTMAX-5      60) SIGRTMAX-4        61) SIGRTMAX-3        62) SIGRTMAX-2
63) SIGRTMAX-1     64) SIGRTMAX

```

## 信号处理

每个信号都有一个当前处理，即当收到信息时，执行什么处理。上面表格中的`Action`表示系统 默认的处理行为。

* `Term`: 默认操作是终止该过程。
* `Ign`:默认操作是忽略信号。
* `Core`:默认操作是终止进程并转储核心。
* `Stop`:默认操作是停止该过程。
* `Cont`:如果当前已停止，则默认操作是继续该过程。

进程可以通过 [sigaction](http://man7.org/linux/man-pages/man2/sigaction.2.html)或者[signal](http://man7.org/linux/man-pages/man2/signal.2.html)修改信号处理。  
信号的处理是每个进程的属性：在多线程应用程序中，信号的处理对于所有线程都是相同的。通过[fork](http://man7.org/linux/man-pages/man2/fork.2.html)创建的子进程继承其父进程的信息处理。

## 参考链接

[SIGNAL(7) - Linux Programmer's Manual](http://man7.org/linux/man-pages/man7/signal.7.html)
