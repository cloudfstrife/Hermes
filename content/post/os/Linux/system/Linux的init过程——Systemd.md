---
title: "Linux的init过程——Systemd"
date: 2020-01-14T18:20:44+08:00
categories:
- Linux
- system
tags:
- linux
- init
- Systemd
keywords:
- linux
- init
- Systemd
---

Systemd 是 Linux 操作系统最新的 init 系统，它的主要设计目标是克服 SysVinit 串行执行脚本导致运行效率低的缺点，提高系统的启动速度。同时 Systemd 提供了和 SysVinit 兼容的特性，这降低了系统向 Systemd 迁移的成本。

<!--more-->

## 基本概念

### 单元 (Unit)

系统初始化需要很多操作，比如启动后台服务，挂载文件系统，配置网络等，过程中的每一步都被 Systemd 抽象为一个配置单元，即 **Unit**。

Systemd 的配置单元为以下类型：

| 类型      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| service   | 一个后台服务进程，比如 mysqld，sshd，最常用的一类            |
| socket    | 封装系统或者网络中的一个套接字                               |
| device    | 封装一个存在于 Linux 设备树中的设备                          |
| mount     | 封装文件系统结构层次中的一个挂载点                           |
| automount | 封装系统结构层次中的一个自挂载点                             |
| swap      | 和挂载配置单元类似，交换配置单元用来管理交换分区             |
| target    | 逻辑分组，本身不执行任何操作，仅引用其他配置单元，用于实现类似运行级别 |
| timer     | 定时器配置单元，用来定时触发用户定义的操作                   |
| snapshot  | 与 target 配置单元相似，快照保存了系统当前的运行状态。       |

### 依赖关系

虽然 Systemd 将大量的启动工作解除了依赖，使得它们可以并发启动。但还是存在有些任务，它们之间存在必要的依赖，比如挂载必须等待挂载点在文件系统中被创建。为了解决这类依赖问题，Systemd 的配置单元之间可以定义彼此的依赖关系。Systemd 用配置单元定义文件中的关键字来描述配置单元之间的依赖关系。

### 事务

Systemd 能保证事务完整性。

Systemd 的事务概念和数据库中的事务不同，主要是为了保证多个依赖的配置单元之间没有循环依赖。

配置单元之间的依赖关系有两种：`required` 强依赖，`want` 弱依赖。当出现循环依赖时，Systemd 尝试去除 wants 关键字指定的依赖，看是否可以打破循环，如果无法打破，Systemd 就会报错。

### target 

Systemd 用 target 替代了 SysVinit 的运行级别，并使用 target 提供了更大的灵活性。

## Systemd 初始化过程

内核初始化完成后，启动 `systemd` 进程，`systemd` 开始执行第一个 target : `default.target` 一般情况下，`default.target`是 `graphical.target` 的软连接。位于 `/usr/lib/systemd/system` 目录下。

```text
$ ls -alh /usr/lib/systemd/system/default.target 
lrwxrwxrwx 1 root root 16 10月 16 21:24 /usr/lib/systemd/system/default.target -> graphical.target
```

### graphical.target

```text
$ cat graphical.target
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

### multi-user.target

`graphical.target` 依赖 `multi-user.target` ，为多用户支持设定系统环境

```text
$ cat multi-user.target
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes

```

`multi-user.target` 将子单元放在目录 `/etc/systemd/system/multi-user.target.wants`中

```text
$ ls -alh multi-user.target.wants/
总用量 20K
drwxr-xr-x  2 root root 4.0K 12月  5 10:04 .
drwxr-xr-x 20 root root  16K 12月  6 16:36 ..
lrwxrwxrwx  1 root root   15 6月  10  2019 dbus.service -> ../dbus.service
lrwxrwxrwx  1 root root   15 10月 16 21:24 getty.target -> ../getty.target
lrwxrwxrwx  1 root root   33 10月 16 21:24 systemd-ask-password-wall.path -> ../systemd-ask-password-wall.path
lrwxrwxrwx  1 root root   25 10月 16 21:24 systemd-logind.service -> ../systemd-logind.service
lrwxrwxrwx  1 root root   39 10月 16 21:24 systemd-update-utmp-runlevel.service -> ../systemd-update-utmp-runlevel.service
lrwxrwxrwx  1 root root   32 10月 16 21:24 systemd-user-sessions.service -> ../systemd-user-sessions.service

```

### basic.target 

`multi-user.target` 依赖 `basic.target`

```text
$ cat basic.target 
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Basic System
Documentation=man:systemd.special(7)
Requires=sysinit.target
Wants=sockets.target timers.target paths.target slices.target
After=sysinit.target sockets.target paths.target slices.target tmp.mount

# We support /var, /tmp, /var/tmp, being on NFS, but we don't pull in
# remote-fs.target by default, hence pull them in explicitly here. Note that we
# require /var and /var/tmp, but only add a Wants= type dependency on /tmp, as
# we support that unit being masked, and this should not be considered an error.
RequiresMountsFor=/var /var/tmp
Wants=tmp.mount

```

### sysinit.target

`basic.target` 依赖于 `sysinit.target` ， `sysinit.target` 会启动重要的系统服务例如系统挂载，内存交换空间和设备，内核补充选项

```text
$ cat sysinit.target
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=System Initialization
Documentation=man:systemd.special(7)
Conflicts=emergency.service emergency.target
Wants=local-fs.target swap.target
After=local-fs.target swap.target emergency.service emergency.target
```

`sysinit.target` 的子单元位于 `/etc/systemd/system/sysinit.target.wants` ，

```text
$ ls -alh sysinit.target.wants/
总用量 20K
drwxr-xr-x  2 root root 4.0K 12月  5 09:44 .
drwxr-xr-x 20 root root  16K 12月  6 16:36 ..
lrwxrwxrwx  1 root root   20 10月 16 21:24 cryptsetup.target -> ../cryptsetup.target
lrwxrwxrwx  1 root root   22 10月 16 21:24 dev-hugepages.mount -> ../dev-hugepages.mount
lrwxrwxrwx  1 root root   19 10月 16 21:24 dev-mqueue.mount -> ../dev-mqueue.mount
lrwxrwxrwx  1 root root   28 10月 16 21:24 kmod-static-nodes.service -> ../kmod-static-nodes.service
lrwxrwxrwx  1 root root   36 10月 16 21:24 proc-sys-fs-binfmt_misc.automount -> ../proc-sys-fs-binfmt_misc.automount
lrwxrwxrwx  1 root root   32 10月 16 21:24 sys-fs-fuse-connections.mount -> ../sys-fs-fuse-connections.mount
lrwxrwxrwx  1 root root   26 10月 16 21:24 sys-kernel-config.mount -> ../sys-kernel-config.mount
lrwxrwxrwx  1 root root   25 10月 16 21:24 sys-kernel-debug.mount -> ../sys-kernel-debug.mount
lrwxrwxrwx  1 root root   36 10月 16 21:24 systemd-ask-password-console.path -> ../systemd-ask-password-console.path
lrwxrwxrwx  1 root root   25 10月 16 21:24 systemd-binfmt.service -> ../systemd-binfmt.service
lrwxrwxrwx  1 root root   30 10月 16 21:24 systemd-hwdb-update.service -> ../systemd-hwdb-update.service
lrwxrwxrwx  1 root root   27 10月 16 21:24 systemd-journald.service -> ../systemd-journald.service
lrwxrwxrwx  1 root root   32 10月 16 21:24 systemd-journal-flush.service -> ../systemd-journal-flush.service
lrwxrwxrwx  1 root root   36 10月 16 21:24 systemd-machine-id-commit.service -> ../systemd-machine-id-commit.service
lrwxrwxrwx  1 root root   31 10月 16 21:24 systemd-modules-load.service -> ../systemd-modules-load.service
lrwxrwxrwx  1 root root   30 10月 16 21:24 systemd-random-seed.service -> ../systemd-random-seed.service
lrwxrwxrwx  1 root root   25 10月 16 21:24 systemd-sysctl.service -> ../systemd-sysctl.service
lrwxrwxrwx  1 root root   27 10月 16 21:24 systemd-sysusers.service -> ../systemd-sysusers.service
lrwxrwxrwx  1 root root   37 10月 16 21:24 systemd-tmpfiles-setup-dev.service -> ../systemd-tmpfiles-setup-dev.service
lrwxrwxrwx  1 root root   33 10月 16 21:24 systemd-tmpfiles-setup.service -> ../systemd-tmpfiles-setup.service
lrwxrwxrwx  1 root root   24 10月 16 21:24 systemd-udevd.service -> ../systemd-udevd.service
lrwxrwxrwx  1 root root   31 10月 16 21:24 systemd-udev-trigger.service -> ../systemd-udev-trigger.service
lrwxrwxrwx  1 root root   30 10月 16 21:24 systemd-update-utmp.service -> ../systemd-update-utmp.service

```

### local-fs.target

`local-fs.target` 根据 `/etc/fstab` 和 `/etc/inittab` 来执行相关操作

大致的 target 依赖关系如下 

![](/images/linux/system/systemd/linux-boot-systemd.svg)


## Unit 管理

`systemctl` 是 systemd 的主命令，主要用于管理 Unit，常用操作如下 ：

列出正在运行的 Unit

```text
$ sudo systemctl list-units
```

输出 Unit 的所有属性

```text
$ sudo systemctl show Unit名称
$ sudo systemctl show -p 属性名称 Unit名称
```

启动 Unit

```text
$ sudo systemctl start Unit名称
```

停止 Unit

```text
$ sudo systemctl stop Unit名称
```

重启Unit

```text
$ sudo systemctl restart Unit名称
```

输出 Unit 运行状态

```text
$ sudo systemctl status Unit名称
```

启动时自动激活 Unit

```text
$ sudo systemctl enable Unit名称
```

禁止开机自动激活 Unit

```text
$ sudo systemctl disable Unit名称
```

查看配置文件

```text
$ sudo systemctl cat Unit名称
```

获取默认启动的target

```text
$ sudo systemctl get-default
```

列出指定target依赖的单元

```text
$ sudo systemctl list-dependencies target名称
```

修改配置文件以后重新加载配置文件

```text
$ sudo systemctl daemon-reload
```
