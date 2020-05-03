---
title: "Linux命令：trap"
date: 2019-12-31T19:47:47+08:00
categories:
- linux
- command
tags:
- linux
- command
- trap
keywords:
- linux
- command
- trap
---

`trap`命令用于指定接收到系统信号后将要采取的动作，常见的用途是在脚本程序被中断时完成清理工作。

<!--more-->

## 用法

```text
$ trap --help
trap: trap [-lp] [[参数] 信号声明 ...]
    对信号和其他事件设陷阱。
    
    定义一个处理器，在 shell 接收到信号和其他条件下执行。
    
    ARG 参数是当 shell 接收到 SIGNAL_SPEC 信号时读取和执行的命令。
    如果没有指定 ARG 参数 (并且只给出一个 SIGNAL_SPEC 信号) 或者ARG 参数为`-`，每一个指定的参数会被重置为原始值。如果 ARG 参数是一个空串，则每一个SIGNAL_SPEC 信号会被 shell 和它启动的命令忽略。
    
    如果一个 SIGNAL_SPEC 信号是 EXIT (0) ，则 ARG 命令会在 shell 退出时被执行。如果一个 SIGNAL_SPEC 信号是 DEBUG，则 ARG命令会在每一个简单命令之前执行。
    
    如果不提供参数，trap 打印列表显示每一个与每一个信号相关联的命令。
    
    选项：
      -l	打印一个信号名称和它们对应的编号的列表
      -p	打印与每个 SIGNAL_SPEC 信号相关联的陷阱命令
    
    每一个 SIGNAL_SPEC 信号可以是 <signal.h> 中的信号名称或者信号编号。信号名称大小写敏感且可以使用 SIG 前缀。信号可用 "kill -信号 $$"发送给 shell。
    
    退出状态：
    返回成功，除非使用了无效的选项或者 SIGSPEC。
```

支持的信号

```text
$ trap -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX	

```

## 示例

在 shell 中清除临时文件

```bash
#!/bin/bash

trap 'rm -f "$TMPFILE"' EXIT

TMPFILE=$(mktemp) || exit 1

...

```

上面的 shell ，无论最终脚本正常结束，还是用户按 `Ctrl + C` 提前终止执行，都会产生 `EXIT` 信号，从而触发trap设置的行动，删除临时文件。


## 小技巧

如果 `trap` 需要执行多条命令，可以封装一个函数

```bash
function trap_func {
  command1
  command2
  ...
}

trap trap_func SIGNAL_SPEC
```
