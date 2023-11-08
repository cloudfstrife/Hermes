---
title: "Linux命令：screenfetch"
date: 2021-03-05T11:40:39+08:00
categories:
- linux
- command
tags:
- linux
- command
- screenfetch
keywords:
- linux
- command
- screenfetch
---

`screenfetch` 是一个获取和显示系统软硬件，主题信息的命令。它可以在 Linux，OS X，FreeBSD 以及其它的类Unix系统上使用。

<!--more-->

## 安装

Debian ＆ ubuntu

```text
sudo apt-get install screenfetch
```

## 运行效果

![ubuntu](/images/linux/screenfetch/01.png)

不同系统的输出并不相同，右侧自上而下的信息包含：用户名与主机名，操作系统及版本号，操作系统核心版本号，系统运行时长，安装的软件包数量，Shell名称及版本，屏幕分辨率，桌面环境，窗口管理器程序，窗口管理器的主题，GTK主题，图标集，默认字体，硬盘空间大小及使用率，CPU基本信息，GPU基本信息，内存信息。

**常用的选项**

`-n` : 隐藏 ASCII LOGO  
`-L` : 只显示 ASCII LOGO  
`-p` : 纵向输出  
`-s` : 输出后自动截图  
`-u` : 此选项只与-s联用，用于截图后，上传至预设置的图片存储服务，可选：`teknik`, `imgur`, `mediacrush`, `hmp`  

## screenfetch 的作用

`screenfetch` 最大的作用可能就是用于在论坛提问时描述自己的环境信息，尤其是桌面应用。在提问时，附带上自己的环境信息将有助于开发者快速定位问题。