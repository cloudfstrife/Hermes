---
title: "Linux审查登陆记录"
date: 2019-11-06T17:59:17+08:00
categories:
- linux
tags:
- linux
- log
- login
keywords:
- linux
- log
- login
---

在Linux系统中，登录日志主要存储在三个文件中： `/var/log/wtmp`,`/var/run/utmp`,`/var/log/lastlog` 。这些文件都是二进制文件，需要使用其它命令来查看登录信息

<!--more-->

常用的查询命令有 `w`,`who`,`users`,`last`,`lastb`,`lastlog` 等。

## w

`w` 命令可用于显示当前登录系统的用户信息。

```text
 15:06:41 up  1:18,  1 user,  load average: 0.08, 0.03, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
xxxxx    pts/0    xxx.xxx.xxx.xxx  13:49    0.00s  2.05s  0.00s w
```

`w`命令显示的对应信息如下：

```text
当前时间 系统启动到现在的时长 登录用户的数目 系统在最近1秒/5秒/15秒的平均负载  
USER: 登录帐号  
TTY : 终端名称  
FROM: 远程主机名  
LOGIN@: 登录时间  
IDLE: 空闲时间  
JCPU: 该TTY终端连接的所有进程的占用时间  
PCPU: 当前进程(即w项中显示的)的占用时间  
WHAT: 当前正在运行进程的命令行  
```

## who

`who` 命令用于显示当前当登录的用户的信息

```text
$ who
xxxxx    pts/0        2019-11-05 13:49 (xxx.xxx.xxx.xxx)
```

`who`命令显示的对应信息如下：

```text
登录帐号 终端名称 登录时间 用户登录IP地址
```

## users

`users` 命令用于显示当前当登录的用户的用户名。如果一个用户有多个登录会话，则用户名将显示多次。

```text
$ users
xxxxx yyyyy
```

## last

`last` 用于显示用户最近登录信息。单独执行`last`命令，它会读取`/var/log/wtmp`的文件，并把该给文件的内容记录的登入系统的用户名单全部显示出来。

```text
$ last -n 10
xxxxx    pts/0        xxx.xxx.xxx.xxx  Tue Nov  5 13:49   still logged in
reboot   system boot  4.19.0-6-amd64   Tue Nov  5 13:48   still running
xxxxx    pts/0        xxx.xxx.xxx.xxx  Mon Nov  4 10:23 - 11:49  (01:25)
reboot   system boot  4.19.0-6-amd64   Mon Nov  4 10:21 - 11:49  (01:27)
xxxxx    pts/0        xxx.xxx.xxx.xxx  Wed Oct 30 10:19 - 16:25  (06:06)
reboot   system boot  4.19.0-6-amd64   Wed Oct 30 10:18 - 16:25  (06:07)
xxxxx    pts/0        xxx.xxx.xxx.xxx  Tue Oct 29 10:44 - 10:44  (00:00)
xxxxx    pts/0        xxx.xxx.xxx.xxx  Tue Oct 29 10:12 - 10:44  (00:31)
reboot   system boot  4.19.0-6-amd64   Tue Oct 29 10:12 - 10:44  (00:32)
xxxxx    pts/0        xxx.xxx.xxx.xxx  Wed Oct 16 18:10 - 18:14  (00:04)

wtmp begins Fri Jul 26 10:06:12 2019
```

`last`命令显示的对应信息如下：

```text
用户名称 终端名称 用户登录IP地址 日志活动发生时间 (括号中的数字表示连接持续了多少小时和分钟)
```

## lastb

`lastb` 命令用于显示用户错误的登录列表，此指令可以发现系统的登录异常。单独执行`lastb`命令，它会读取位于`/var/log/btmp`的文件，并把该文件内容记录的登入失败的用户名单，全部显示出来。

```text
$ sudo lastb

btmp begins Mon Nov  4 10:21:35 2019
```

## lastlog 

`lastlog` 命令用于显示系统中所有用户最近一次登录信息。

```text
用户名           端口     来自             最后登陆时间
root                                       **从未登录过**
daemon                                     **从未登录过**
bin                                        **从未登录过**
sys                                        **从未登录过**
sync                                       **从未登录过**
games                                      **从未登录过**
man                                        **从未登录过**
lp                                         **从未登录过**
mail                                       **从未登录过**
news                                       **从未登录过**
uucp                                       **从未登录过**
proxy                                      **从未登录过**
www-data                                   **从未登录过**
backup                                     **从未登录过**
list                                       **从未登录过**
irc                                        **从未登录过**
gnats                                      **从未登录过**
nobody                                     **从未登录过**
_apt                                       **从未登录过**
systemd-timesync                           **从未登录过**
systemd-network                            **从未登录过**
systemd-resolve                            **从未登录过**
messagebus                                 **从未登录过**
xxxxx            pts/0    192.168.88.1     二 11月  5 13:49:00 +0800 2019
systemd-coredump                           **从未登录过**
sshd                                       **从未登录过**
```
