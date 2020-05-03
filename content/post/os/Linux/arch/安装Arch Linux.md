---
title: "安装Arch Linux"
date: 2019-04-09T12:08:06+08:00
categories:
- Linux
- Arch
tags:
- Linux
- Arch
keywords:
- Linux
- Arch
---

虚拟机安装 Arch Linux 的过程

<!--more-->


## 下载与刻录

略

## 使用光盘启动

略

## 硬盘分区

```text
parted --align optimal /dev/sda	

mklab gpt
mkpart primary 0% 500
mkpart primary 500 4500
mkpart primary 4500 11475
mkpart primary 11475 100%
name 1 EFI
name 2 swap
name 3 base
name 4 data
set 1 boot on
```

> 这里硬盘分四个区 

## 格式化分区

```text
mkfs.vfat -F32 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
```

## 挂载分区

```text
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mkdir -p /mnt/data
mount /dev/sda1 /mnt/boot/efi
mount /dev/sda4 /mnt/data
swapon /dev/sda2
```

> 注意这里的顺序，先挂载根分区(/)，其它分区挂在根分区下面

## 修改软件源镜像服务器

```text
vim /etc/pacman.d/mirrorlist
```
> 注释掉所有Server 

使用vim替换命令

```text
:.,$s/Server /# Server /g
```
> 打开China镜像

使用vim查找，并删除掉前面的注释符(#)

```text
/China
```

## 更新软件源

```text
pacman -Syy
```

## 安装基础系统

```text
pacstrap /mnt base base-devel
```

## 生成fstab

```text
genfstab -U /mnt >> /mnt/etc/fstab 
```

## 进入新系统

```text
arch-chroot /mnt
```

## 设置locale

```text
vim /etc/locale.gen
```

> 去除`en_US.UTF-8 UTF-8`和`zh_CN.UTF-8 UTF-8`前的注释符(#)

```text
echo "LANG=zh_CN.UTF-8" > /etc/locale.conf
locale-gen zh_CN.UTF-8
```

## 设置主机名

```text
echo "主机名" > /etc/hostname

vim /etc/hosts
```

> 在文件末尾添加如下内容：

```text
127.0.0.1	localhost
::1			localhost
```


## 设置自动配置网络

```text
systemctl enable dhcpcd
```

## 新增用户

```text
useradd -m -g users -s /bin/bash username
passwd username
```

### 配置sudo

```text
pacman -S sudo
chmod 755 /etc/sudoers
vim /etc/sudoers
chmod 440 /etc/sudoers
```

> 复制`root ALL=(ALL) ALL` 改为 `username ALL=(ALL) ALL`

## 安装GRUB

```text
pacman -S grub-efi-x86_64 efibootmgr os-prober 

grub-install --efi-directory=/boot/efi --bootloader-id=grub --bootloader-id=grub --recheck

grub-mkconfig -o /boot/grub/grub.cfg
```

## 退出

```text
exit

umount /dev/sda4
umount /dev/sda1
umount /dev/sda3

shutdown -h now 
```

## 重启测试

略
