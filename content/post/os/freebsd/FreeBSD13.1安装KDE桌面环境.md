---
title: "FreeBSD 13.1 安装 KDE 桌面环境"
date: 2022-11-06T16:11:05+08:00
categories:
- FreeBSD
tags:
- FreeBSD
- KDE
keywords:
- FreeBSD
- KDE
---

记录 FreeBSD 安装 KDE 桌面环境的过程 

<!--more-->

## 安装完成的后处理

### 添加 proc 分区
```text
sudo vim /etc/fstab

# 增加
proc /proc procfs rw 0 0
```

### wifi 驱动

> 这里以 intel AC 9260 为例

```text
cat << EOF | sudo tee -a /boot/loader.conf

if_iwm_load="YES"
iwm3160fw_load="YES"
iwm3168fw_load="YES"
iwm7260fw_load="YES"
iwm7265fw_load="YES"
iwm8000Cfw_load="YES"
iwm8265fw_load="YES"
iwm9000fw_load="YES"
iwm9260fw_load="YES"
EOF
```

### 修改 pkg 镜像源

```text
mkdir -p /usr/local/etc/pkg/repos/
cp /etc/pkg/FreeBSD.conf /usr/local/etc/pkg/repos/

vi /usr/local/etc/pkg/repos/FreeBSD.conf

# 内容如下

FreeBSD: {
  url: "pkg+http://mirrors.ustc.edu.cn/freebsd-pkg/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}

```

### 安装 sudo 并配置 wheel 用户组可用

```text
pkg install sudo 
/usr/local/sbin/visudo

# 去除 wheel 组配置前面的 #
%wheel ALL=(ALL:ALL) ALL
```

## 更新操作系统

```text
sudo freebsd-update fetch
sudo freebsd-update install
```

## 更新 pkg 软件信息

```text
sudo pkg update
sudo pkg upgrade
```

## 安装基础软件
```text
sudo pkg install vim wqy-fonts
```

## 安装驱动

> 以 intel 集成显卡为例

```text
sudo pkg install xf86-video-intel xf86-input-synaptics drm-kmod webcamd
```

### 添加到 video 组
```text
sudo pw groupmod video -m root,cloud
sudo pw groupmod webcamd -m root,cloud
```

### 加载核心模块 cuse 

> webcamd 需要 cuse

```text
cat << EOF | sudo tee -a /boot/loader.conf
cuse_load="YES"
EOF
```

###  启动 evdev
```text
sudo sysctl kern.evdev.rcpt_mask=6
```

### 启用组件

```text
sudo sysrc webcamd_enable=YES
sudo sysrc kld_list=i915kms
```

## 安装 xorg 
```text
sudo pkg install xorg 
```

### 生成 xorg 配置文件

```text
sudo Xorg -configure
mv /root/xorg.conf.new /etc/X11/xorg.conf
```

## 安装 sddm plasma5

```text
sudo pkg install sddm plasma5-plasma plasma5-sddm-kcm
```

### 内核参数

```text
sudo sysctl net.local.stream.recvspace=65536
sudo sysctl net.local.stream.sendspace=65536
```

### 启动 dbus sddm
```text
sudo sysrc dbus_enable=YES
sudo sysrc sddm_enable=YES
```

### 添加 startplasma-x11 启动项

```text
echo "exec ck-launch-session startplasma-x11" | tee -a ~/.xinitrc
echo "exec ck-launch-session startplasma-x11" | sudo tee -a /root/.xinitrc
```

### 添加 polkit 规则

```text
cat << EOF | tee -a /usr/local/etc/polkit-1/rules-d/40-wheel-group.rules
polkit.addRule(function(action, subject) {
    if (subject.isInGroup("wheel")) {
    	return polkit.Result.YES;
    }
});
EOF
```

## 安装软件

```text
sudo pkg install konsole dolphin dolphin-plugins firefox-esr
```

## 安装输入法

```text
sudo pkg install fcitx5 fcitx5-configtool fcitx5-gtk fcitx5-qt zh-fcitx5-chinese-addons

cat << EOF | sudo tee -a /etc/profile

export XMODIFIERS='@im=fcitx5'
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
EOF

mkdir -p ~/.config/autostart/
cp /usr/local/share/applications/org.fcitx.Fcitx5.desktop  ~/.config/autostart/
```

## zsh

```text
sudo pkg install zsh zsh-antigen zsh-autosuggestions zsh-navigation-tools zsh-syntax-highlighting ohmyzsh

cp /usr/local/share/ohmyzsh/templates/zshrc.zsh-template ~/.zshrc

chsh -s zshrc
```

### 如果 xorg 下箭头键不好用可以启动这些环境变量
```text
export XKB_DEFAULT_RULES=xorg
setenv XKB_DEFAULT_RULES xorg
```

### xserver 可以使用文泉驿字体

```text
sudo xset fp+ /usr/local/share/fonts/wqy
sudo xset fp rehash
```
