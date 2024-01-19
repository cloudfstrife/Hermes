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

### 分区

```text
parted --align optimal /dev/sda	

mklabel gpt
mkpart ESP fat32 0% 512
mkpart primary 512 1024
mkpart primary 1024 100%
name 1 EFI
name 2 boot
name 3 base
set 1 boot on
set 3 lvm on
```

### 加密

```text
cryptsetup --verbose --cipher=aes-xts-plain64 --key-size=512 --hash=sha512 --use-random luksFormat /dev/sda3
```

### 解锁加密分区

```text
cryptsetup open --type luks /dev/sda3 base
```

### 创建LVM逻辑卷

```text
## 创建物理卷
pvcreate /dev/mapper/base

## 创建卷组
vgcreate base /dev/mapper/base

## 创建逻辑卷
lvcreate -n swap -L 70G base
lvcreate -n base -L 120G base
lvcreate -n swap -l 100%FREE data
```

### 格式化分区

```text
mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/mapper/base-base
mkfs.ext4 /dev/mapper/base-data
mkswap /dev/mapper/base-swap
```

### 挂载分区

```text
mount /dev/mapper/base-base /mnt

mkdir -p /mnt/boot
mount /dev/sda2 /mnt/boot

mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi

mkdir -p /mnt/data
mount /dev/mapper/base-data /mnt/data

swapon /dev/mapper/base-swap
swapon -a 
swapon -s

lsblk -f
```

> 注意这里的顺序，先挂载根分区(/)，其它分区挂在根分区下面

## 安装基本系统

### 修改软件源镜像服务器

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

### 更新软件源

```text
pacman -Syy
```

### 安装基础系统

```text
pacstrap /mnt base base-devel linux linux-firmware grub efibootmgr dialog xterm vim networkmanager
```

### 生成fstab

```text
genfstab -U /mnt >> /mnt/etc/fstab 
```

## 配置 

### chroot

```text
arch-chroot /mnt
```

### locale

```text
vim /etc/locale.gen
```

> 去除`en_US.UTF-8 UTF-8`和`zh_CN.UTF-8 UTF-8`前的注释符(#)

```text
locale-gen zh_CN.UTF-8
```

### 语言环境

```text
echo "LANG=zh_CN.UTF-8" > /etc/locale.conf
export LANG=zh_CN.UTF-8
```

### 时区

```text
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 主机名

```text
echo "主机名" > /etc/hostname
```

### hosts

```text
cat << EOF | tee -a  /etc/hosts
127.0.0.1	localhost
::1			localhost
EOF
```

### 新增用户

```text
useradd -m -G wheel,games,power,optical,scanner,lp,audio,video -s /bin/bash username
passwd username
```

### 配置sudo

```text
pacman -S sudo
visudo
```

> 打开 `%wheel ALL=(ALL) ALL` 的注释

### 配置内核参数

```text
vim /etc/default/grub
```

> 修改 `GRUB_CMDLINE_LINUX_DEFAULT` ，增加 `quiet resume=/dev/base/swap cryptdevice=/dev/sda3:base`，完整行如下：

> ```text
> GRUB_CMDLINE_LINUX="quiet resume=/dev/base/swap cryptdevice=/dev/nvme0n1p3:base"
> ```

### 配置 mkinicpio

```text
vim /etc/mkinitcpio.conf 
```
> 修改 `HOOKS` ，增加 `encrypt lvm2`，完整行如下：

```text
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```

### 配置 vconsole

```text
cat << EOF | tee -a /etc/vconsole.conf
KEYMAP=us
FONT=lat9u-16
EOF
```

### 生成 initramfs 映像

```text
mkinitcpio -p linux
```

### 安装GRUB

```text
pacman -S grub-efi-x86_64 efibootmgr os-prober 
grub-install --efi-directory=/boot/efi --bootloader-id=archlinu --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

### 安装 KDE

```text
pacman -S plasma plasma-wayland-session wqy-microhei konsole ark dolphin dolphin-plugins noto-fonts-cjk noto-fonts-emoji
systemctl enable sddm.service
systemctl enable NetworkManager.service
```

## 退出

```text
exit

umount /dev/sda1
umount /dev/sda2
umount /dev/mapper/base-base
umount /dev/mapper/base-data
swapoff /dev/mapper/base-swap

cryptsetup close /dev/mapper/base-base
cryptsetup close /dev/mapper/base-data
cryptsetup close /dev/mapper/base-swap
cryptsetup close /dev/mapper/base

reboot
```

## 重启测试

略
