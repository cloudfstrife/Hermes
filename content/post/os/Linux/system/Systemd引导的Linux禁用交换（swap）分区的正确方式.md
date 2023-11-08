---
title: "Systemd引导的Linux禁用交换（swap）分区的正确方式"
date: 2020-07-12T18:10:02+08:00
categories:
- Linux
- system
tags:
- Linux
- system
- swap
- 交换分区
keywords:
- Linux
- system
- swap
- 交换分区
---

安装 kubernetes 过程中，要禁用 swap 分区，按照网上的教程操作完成之后，重启服务器，swap 分区依然挂载了，针对这个问题，将 systemd Init系统禁用交换分区的操作操作记录下来备忘

<!--more-->

## SysVinit 禁用交换分区的方式

### 查看交换分区状态

```text
$ sudo swapon --show
```

### 临时禁用

```text
$ sudo swapoff -a 
```

### 永久禁用

编辑 `/etc/fstab` 文件，注释 swap 类型的行

```text
$ cat -n /etc/fstab 
     1	# /etc/fstab: static file system information.
     2	#
     3	# Use 'blkid' to print the universally unique identifier for a
     4	# device; this may be used with UUID= as a more robust way to name devices
     5	# that works even if disks are added and removed. See fstab(5).
     6	#
     7	# <file system> <mount point>   <type>  <options>       <dump>  <pass>
     8	# / was on /dev/sda4 during installation
     9	UUID=20ba574c-bb7d-476f-9b56-d443d7181199 /               ext4    errors=remount-ro 0       1
    10	# /boot was on /dev/sda2 during installation
    11	UUID=e1d047ba-8b90-48b0-95cc-203566c26182 /boot           ext4    defaults        0       2
    12	# /boot/efi was on /dev/sda1 during installation
    13	UUID=2490-C1A4  /boot/efi       vfat    umask=0077      0       1
    14	# /data was on /dev/sda5 during installation
    15	UUID=690cb647-3c8c-4bbd-b61c-fc55b7d53d96 /data           ext4    defaults        0       2
    16	# swap was on /dev/sda3 during installation
    17	# UUID=561c1ccf-96e8-49d2-bd2f-051d262fd6bf none            swap    sw              0       0
    18	/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0

```
> 本例第17行为注释掉的 swap 分区

执行以上操作之后，重启系统发现 swap 分区依然被开启了

## systemd Init 系统禁用交换分区的方式

```text
$ sudo systemctl mask dev-sdXX.swap
```

> `dev-sdXX.swap` 中的 XX 为 swap 分区

**示例**

```text
$ sudo systemctl mask dev-sda3.swap 
Created symlink /etc/systemd/system/dev-sda3.swap → /dev/null.
```

## 参考链接

[Best way to disable swap in Linux](https://serverfault.com/questions/684771/best-way-to-disable-swap-in-linux)