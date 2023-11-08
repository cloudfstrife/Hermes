---
title: "fstab文件"
date: 2019-02-20T12:07:47+08:00
categories:
- Linux
tags:
- Linux
- fstab
keywords:
- Linux
- fstab
---

`/etc/fstab`记录Linux操作系统的静态文件系统配置信息。当Linux系统启动的时候，系统会自动读取文件信息，将此文件中指定的文件系统挂载到指定的目录。

<!--more-->

## fatab的作用

`fstab`文件可用于定义磁盘分区、各种其他块设备或远程文件系统应如何装入文件系统。

## 示例

下面是一个简单的示例：

```text
$ cat /etc/fstab 
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mapper/base-base /               ext4    errors=remount-ro 0       1
# /boot was on /dev/sda2 during installation
UUID=b0dacfa8-d34f-44f7-92fa-7e35fa932dec /boot           ext4    defaults        0       2
# /boot/efi was on /dev/sda1 during installation
UUID=3D70-82F8  /boot/efi       vfat    umask=0077      0       1
/dev/mapper/base-data /data           ext4    defaults        0       2
/dev/mapper/swap-swap none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
```

> 因为启用了LVM所以这个文件看上去和一般的文件不同。`/dev/mapper/`开头的文件系统是LVM的逻辑卷。

`/etc/fstab`文件第一行包含以下几个字段，每个字段使用空格或者tab分隔

```text
<file system> <mount point>   <type>  <options>       <dump>  <pass>
```

* <file systems>      &nbsp;&nbsp;&nbsp;&nbsp;要挂载的分区或设备
* <mount point>       &nbsp;&nbsp;&nbsp;&nbsp;挂载点，对于swap分区，这个域应该填写：none，表示没有挂载点
* <type>              &nbsp;&nbsp;&nbsp;&nbsp;挂载文件系统类型
* <options>           &nbsp;&nbsp;&nbsp;&nbsp;挂载时使用的参数
* <dump>              &nbsp;&nbsp;&nbsp;&nbsp;`dump`程序检查此字段内容，并用数字来决定是否对这个文件系统进行备份。允许的数字是 0 和 1。0表示忽略， 1则进行备份。
* <pass>              &nbsp;&nbsp;&nbsp;&nbsp;用来指定fsck检查硬盘的顺序，允许的数字是0,1,和2。如果这里填0，则不检查。`/`(根分区)挂载点必须填写1，其他的分区可以填0或者1

## file systems

在fatab文件中，文件系统标识可以使用三种不同的方式表示，**内核名称**、**UUID**、**label**

### 内核名称

使用命令`sudo fdisk -l `来获取设备的内核名称

```text
$ sudo fdisk -l /dev/sda
Disk /dev/sda: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 69489558-561A-473E-A071-80793EBA1145

Device       Start      End  Sectors  Size Type
/dev/sda1     2048   391167   389120  190M EFI System
/dev/sda2   391168  1368063   976896  477M Linux filesystem
/dev/sda3  1368064  5273599  3905536  1.9G Linux LVM
/dev/sda4  5273600 83884031 78610432 37.5G Linux LVM
```

### label

> 使用这一方法，每一个标签必须是唯一的

要显示所有设备的标签，可以使用`lsblk -f`命令。

```text
$ sudo lsblk -f
NAME          FSTYPE      LABEL UUID                                   MOUNTPOINT
sda                                                                    
├─sda1        vfat              3D70-82F8                              /boot/efi
├─sda2        ext4              b0dacfa8-d34f-44f7-92fa-7e35fa932dec   /boot
├─sda3        LVM2_member       Ty6gmx-MmaJ-MGqN-SYvJ-cIDX-WFYb-UrF3tv 
│ └─swap-swap swap              0295efa6-02c8-48a6-ba0a-2c803aadfd4e   [SWAP]
└─sda4        LVM2_member       rgOU8C-tIfw-2ZNM-URID-4B9f-TR7Z-pb5S55 
  ├─base-base ext4              2eaf31d6-924b-4351-bc06-051d02c71a40   /
  └─base-data ext4              cf381daa-c1bc-422d-b316-037cb1a7a37f   /data

```

在`/etc/fstab`中使用`LABEL=`作为设备名的开头

```text
/etc/fstab

# <file system>        <dir>         <type>    <options>             <dump> <pass>

tmpfs                  /tmp          tmpfs     nodev,nosuid   0      0
 
LABEL=Arch_Linux       /             ext4      defaults,noatime      0      1
LABEL=Arch_Swap        none          swap      defaults              0      0
```

### UUID

所有分区和设备都有唯一的 UUID。它们由文件系统生成工具 (mkfs.*) 在创建文件系统时生成。 

`lsblk -f`命令可以查看设备的UUID值。`/etc/fstab`中使用`UUID=`前缀

```text
/etc/fstab

# <file system>                           <dir>         <type>    <options>             <dump> <pass>

tmpfs                                     /tmp          tmpfs     nodev,nosuid          0      0
 
UUID=24f28fc6-717e-4bcd-a5f7-32b959024e26 /     ext4              defaults,noatime      0      1
UUID=03ec5dd3-45c0-4f95-a363-61ff321a09ff /home ext4              defaults,noatime      0      2
UUID=4209c845-f495-4c43-8a03-5363dd433153 none  swap              defaults              0      0
```

## type

要挂载设备或是分区的文件系统类型，支持许多种不同的文件系统：`ext2`, `ext3`, `ext4`, `reiserfs`, `xfs`, `jfs`, `smbfs`, `iso9660`, `vfat`, `ntfs`, `swap` 及 `auto`。 

设置成`auto`类型，`mount` 命令会猜测使用的文件系统类型，对 `CDROM` 和 `DVD` 等移动设备是非常有用的。

## options

`options`指定了一系列挂载时使用的参数，有些参数是特定文件系统才有的。

* `auto` - 在启动时或键入了 `mount -a` 命令时自动挂载。
* `noauto` - 只在你的命令下被挂载。
* `exec` - 允许执行此分区的二进制文件。
* `noexec` - 不允许执行此文件系统上的二进制文件。
* `ro` - 以只读模式挂载文件系统。
* `rw` - 以读写模式挂载文件系统。
* `user` - 允许任意用户挂载此文件系统，若无显示定义，隐含启用 `noexec`, `nosuid`, `nodev` 参数。
* `users` - 允许所有 `users` 组中的用户挂载文件系统.
* `nouser` - 只能被 `root` 挂载。
* `owner` - 允许设备所有者挂载.
* `sync` - I/O 同步进行。
* `async` - I/O 异步进行。
* `dev` - 解析文件系统上的块特殊设备。
* `nodev` - 不解析文件系统上的块特殊设备。
* `suid` - 允许 `suid` 操作和设定 `sgid` 位。这一参数通常用于一些特殊任务，使一般用户运行程序时临时提升权限。
* `nosuid` - 禁止 suid 操作和设定 sgid 位。
* `noatime` - 不更新文件系统上 `inode` 访问记录，可以提升性能(参见 `atime` 参数)。
* `nodiratime` - 不更新文件系统上的目录 inode 访问记录，可以提升性能(参见 `atime` 参数)。
* `relatime` - 实时更新 inode access 记录。只有在记录中的访问时间早于当前访问才会被更新。（与 noatime 相似，但不会打断如 mutt 或其它程序探测文件在上次访问后是否被修改的进程。），可以提升性能(参见 atime 参数)。
* `flush` - vfat 的选项，更频繁的刷新数据，复制对话框或进度条在全部数据都写入后才消失。
* `defaults` - 使用文件系统的默认挂载参数，例如 `ext4` 的默认参数为:`rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, `async`.

## 文件系统挂载的一些约束

* 根目录必须挂载，而且要先于其他mount point被挂载
* 挂载点必须是已经存在的目录
* 挂载点必须遵守必要的系统目录架构原则
* 一个文件系统同一时间只能挂载一次

## 参考链接

[https://wiki.archlinux.org/index.php/Fstab_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)](https://wiki.archlinux.org/index.php/Fstab_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

