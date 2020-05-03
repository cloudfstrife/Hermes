---
title: "Linux的init过程——SysVinit"
date: 2020-01-13T18:35:49+08:00
categories:
- Linux
- system
tags:
- linux
- init
- SysVinit
keywords:
- linux
- init
- SysVinit
---

Linux 操作系统的启动首先从 BIOS 开始，进而从磁盘加载MBR，接下来进入 bootloader，载入内核，完成内核初始化。内核初始化的最后一步是启动 pid 为 `1` 的**init进程**，这个进程是系统的第一个进程，它负责产生其他用户进程。  
早期的大多数 Linux 发行版的 init 系统是和 System V 相兼容的，被称为 **SysVinit** 。它源于 System V 系列 UNIX 。
<!--more-->

> 注意：目前主流的 Linux 发行版都已经采用 Systemd 的 init系统，SysVinit 只在较旧的版本中可以见到。

## init 程序 

`init` 程序是 SysVinit 的主要程序，位于Linux系统 `/sbin` 目录下

`init` 程序首先读取 `/etc/inittab` 文件，获取系统运行级别 ( runlevel ) 之后跟据运行级别，顺序地执行以下位置的启动脚本，从而将系统初始化为预设的运行级别

* `/etc/rc.d/rc.sysinit`                    &emsp;&emsp;&emsp;&emsp;&emsp; 重要的系统初始化服务
* `/etc/rc.d/rc` 与 `/etc/rc.d/rcX.d/`      &emsp;&emsp;&emsp;&emsp;&emsp; X为运行级别
* `/etc/rc.d/rc.local`                      &emsp;&emsp;&emsp;&emsp;&emsp; 用户个性化服务

## rcX.d 的命令规则

Sysvinit 不仅用于初始化系统，还负责在系统关闭时，执行必要的清理工作。  

在 `/etc/rc.d/rcX.d/` 目录下，文件名以 `S` 开头的脚本是启动时运行的脚本，`S` 后面的数字表示脚本的执行顺序。文件名以 `K` 开头的脚本在关闭系统时运行，字母 `K` 之后的数字表示关闭时的执行顺序。

在 `/etc/rc.d/rcX.d` 目录下的脚本，大部分是软链接，真实的脚本文件一般存放在 `/etc/init.d` 目录下。

整个启动过程如下：

![linux-boot-init](/images/linux/system/sysvinit/linux-boot-init.svg)

## 运行级别

SysVinit 中运行模式描述了系统各种预订的运行模式。  
通常会有 8 种运行模式，即运行模式 0 到 6 和 S 或者 s。

每个 Linux 发行版对运行模式的定义都不太一样。但 `0` , `1` , `6` 一致被约定用于如下模式：

* 0          &emsp;&emsp;关机
* 1          &emsp;&emsp;单用户模式
* 6          &emsp;&emsp;重启

其它的运行模式包括：

* 无网络的多用户
* 命令行模式
* GUI（图形桌面 模式）

S 级别往往用于系统故障之后的排错和恢复。

## rcX.d脚本

`/etc/init.d`目录下的脚本应该是可以接收 `start` , `stop` , `restart` , `try-restart` , `reload` , `force-reload` , `status` 等参数的，至少要可以接收 `start`, `stop` , `restart` , `force-reload`。参见 [writing-the-scripts](https://www.debian.org/doc/debian-policy/ch-opersys.html#writing-the-scripts)

以下是一个模板

```bash
#!/bin/sh
#
# description
# chkconfig: 2345 92 65

### BEGIN INIT INFO
# Provides:          
# Required-Start:    
# Required-Stop:     
# Should-Start:      
# Should-Stop:       
# Default-Start:     
# Default-Stop:      
# Short-Description: 
# Description:       
### END INIT INFO

case "$1" in
start)
	# 执行启动命令
	;;
stop)
	# 执行关闭命令
	;;
restart) 
    # 执行重启命令
	;;
reload|force-reload)
	# 重新加载命令
	;;
status)
    # 查询状态命令
	;;
*)	
	# 其它命令行参数异常处理
	echo "Usage: $N {start|stop|restart|reload|force-reload|status}" >&2
	exit 1
	;;
esac
exit 0

```

> `# chkconfig: 2345 92 65` 是 chkconfig 的指令，定义应在哪个运行级别启用该服务，同时指定了在创建启动和终止脚本的链接时要使用的编号。

`### BEGIN INIT INFO` 到 `### END INIT INFO`之间的注释是运行时依赖注释，块内注释格式为 `# {keyword}: arg1 [arg2...]`

| 名称                | 说明                                         |
| ------------------- | -------------------------------------------- |
| `Provides`          | 定义脚本所提供的服务名称                     |
| `Required-Start`    | 定义脚本需要依赖的服务                       |
| `Required-Stop`     | 定义脚本供哪些服务依赖                       |
| `Should-Start`      | 定义脚本需要依赖的基础服务，非强制依赖       |
| `Should-Stop`       | 定义脚本供哪些服务依赖，非强制依赖           |
| `Default-Start`     | 定义默认情况下应在哪些运行级别启动脚本       |
| `Default-Stop`      | 定义默认情况下应在哪些运行级别停止脚本       |
| `Short-Description` | 短说明                                       |
| `Description`       | 详细说明                                     |

依赖中的一些“虚拟”基础设施。

| 名称         | 说明                                              |
| ------------ | ------------------------------------------------- |
| `$local_fs`  | 所有本地文件系统都已挂载                          |
| `$network`   | 低级联网                                          |
| `$named`     | 主机名解析守护程序已启动                          |
| `$portmap`   | RFC 1833中定义的SunRPC/ONCRPC端口映射服务已运行。 |
| `$remote_fs` | 所有文件系统都已挂载                              |
| `$syslog`    | 系统日志服务                                      |
| `$time`      | 系统时间已设定                                    |
| `$all`       | 所有不依赖于`$all`的脚本                          |

## init 服务管理

启动服务

```text
service 服务名 start 
```

关闭服务

```text
service 服务名 stop
```

重启服务

```text
service 服务名 restart 
```
