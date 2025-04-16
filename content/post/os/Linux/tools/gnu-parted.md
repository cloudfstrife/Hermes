---
title: "GNU Parted"
date: 2018-12-26T18:32:17+08:00
categories:
- Linux
tags:
- Linux
- GNU Parted
keywords:
- Linux
- GNU Parted
---


由于常见的fdisk不支持GPT。所以Linux 命令行模式下，创建和修改GPT分区常常使用GNU Parted之类的工具来完成。

GNU Parted 具有丰富的功能，它除了能够进行分区的添加、删除等常见操作外，还可以进行移动分区、创建文件系统、调整文件系统大小、复制文件系统等操作。它可以处理最常见的分区格式，包括：ext2,ext3,fat16,fat32,NTFS,ReiserFS,JFS,XFS,UFS,HFS,以及Linux交换分区。 

<!--more-->

## GNU parted的使用 

> 以下示例在arch Linux环境下，使用GUN parted 3.2示例

parted 命令的常用格式是：

1. parted [选项] <硬盘设备名>
1. parted [选项] <硬盘设备名> <子命令> [<子命令参数>]

## 常用指令

在命令行模式下，输入`parted /dev/sdx`（其中/dev/sdx为硬盘设备名，如果以普通用户登陆，需要使用sudo或者切换用户为root)。

![](/images/linux/gun_parted/parted_01.png)

### 帮助 

输入`help`

![](/images/linux/gun_parted/parted_02.png)

help指令输出了parted所有的可用指令。

Parted常用指令

| 命令 | 说明 |
|:---|:---|
|help [COMMAND] | 打印命令的帮助信息，或指定命令的帮助信息 |
|print [free|NUMBER|all] | 显示分区表, 指定编号的分区, 或所有设备的分区表 |
|mkpart PART-TYPE [FSTYPE] START END | 创建新分区。PART-TYPE 是以下类型之一：primary（主分 区）、extended（扩展分区）、logical（逻辑分区）。START 和 END 是新分区开始和结束的具体位置。 |
|rm NUMBER | 删除指定编号 NUMBER 的分区。 |
|set NUMBER FLAG STATE | 对指定编号 NUMBER 的分区设置分区标记 FLAG。对于 PC 常用的 msdos 分区表来说，分区标记 FLAG 可有如下值：”boot”（引导）, “hidden”（隐藏）, “raid”（软RAID磁盘阵）, “lvm”（逻辑卷）, “lba” （LBA，Logic Block Addressing模式）。 状态STATE 的取值是：on 或 off |
|unit UNIT | 设置默认输出时表示磁盘大小的单位为 UNIT，UNIT 的常用取值可以为：‘MB’、‘GB’、‘%’（占整个磁盘设备的百分之多少）、‘compact’（人类易读方式，类似于 df 命令中 -h 参数的用）、‘s’（扇区）、‘cyl’ （柱面）、‘chs’ （柱面cylinders:磁头 heads:扇区 sectors 的地址） |
|resizepart NUMBER END | 对指定编号 NUMBER 的分区调整大小。|
|rescue START END |如果你不小心用Parted的rm命令删除了一个分区，那么这个命令可以帮你恢复。你需要给出所误删的分区的大概的开始和结束的位置。Parted 就会在你给出的磁盘区域内去寻找，如果找到这个分区，那么Parted 就会询问你是否重新建立这个分区。|
|mklabel,mktable LABELTYPE | 创建一个新的 LABEL-TYPE 类型的空磁盘分区表，对于PC而言 msdos 是常用的 LABELTYPE。 若是用 GUID 分区表，LABEL-TYPE 应该为 gpt|
|align-check TYPE NUMBER|检查分区性能检查 TYPE取值为minimal,optimal，其中常用的是optimal|


### 分区信息

输入`print`

![](/images/linux/gun_parted/parted_03.png)

### 创建分区表

输入`mklabel gpt`

![](/images/linux/gun_parted/parted_04.png)

上图通过print指令显示创建分区表前后的区别

### 创建分区

输入`mkpart primary 0 200`

![](/images/linux/gun_parted/parted_05.png)

mkpart指令用于创建分区，primary表示创建主分区，0 200 表示起始位置和结束位置。

> mkpart有时会出现警告，如下图，输入i忽略
> ![](/images/linux/gun_parted/parted_08.png)

### 设置磁盘分区单位

输入`unit s`,`print`,`unit MB`,`print`

![](/images/linux/gun_parted/parted_06.png)

* 删除分区

输入`print`,`rm 2`,`print`

![](/images/linux/gun_parted/parted_07.png)

rm指令用于删除分区。

### 修改分区大小

输入`resizepart 1 250`

![](/images/linux/gun_parted/parted_09.png)

resizepart指令用于修改分区大小。

### 为分区命名

输入`name 1 boot`,`print`

![](/images/linux/gun_parted/parted_11.png)

name指令用于命名

### 更改分区标记
 
输入set 1 boot on

![](/images/linux/gun_parted/parted_10.png)

> set指令格式为`set NUMBER FLAG STATUS`。
> 
> NUMBER为分区号
> 
> FLAG取值如下：
> >boot
> >root
> >swap
> >hidden
> >raid
> >lvn
> >lba
> >hp-service
> >palo
> >prep
> >msftres
> >bios\_grub
> >atvrecv
> >diag
> >legacy\_boot
> >msftdata
> >irst
> >esp
> 
> STATUS取值如下：
> >on 
> >off

## 关于4K对齐

当前电脑传统机械硬盘的每个扇区一般大小为512字节；当使用某一文件系统将硬盘格式化时，文件系统会将硬盘扇区、磁道与柱面统计整理并定义一个簇为多少扇区方便快速存储。

如果每个簇都会跨越两个物理单元，占据第一个单元的组后512字节和第二个单元的前3584字节。这样文件系统在读写某个簇的时候，硬盘需要读写两个物理单元，这会降低读写速度，并缩短使用寿命。现时一般使用一些硬盘分区软件在主引导记录的63个扇区后作牺牲地空出数个扇区以对齐文件系统的4096B每簇，以避免过多的读写操作，提升读写速度、延长使用寿命。

当使用parted做mkpart时，产生的警告，就是因为分区未做4K对齐而产生的。

## 使用parted创建做4K对齐的分区表

### step 1
使用`parted --align optimal /dev/sdX`启动parted

### step 2

分区

```text
mklabel gtp
mkpart primary 0% XXX
mkpart primary XXX xxxxx
……
```

> 注意命令第一个分区起始位置使用的是0% 

### setp 3

验证分区是否4K对齐

```text
align-check optimal X
```
