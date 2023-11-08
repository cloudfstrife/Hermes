---
title: "安装FreeBSD"
date: 2019-10-23T18:21:26+08:00
categories:
- FreeBSD
tags:
- FreeBSD
keywords:
- FreeBSD
---

虚拟机安装 FreeBSD 操作系统的过程

<!--more-->

## 新建虚拟机

* 4核心CPU
* 40G硬盘
* 2G内存
* 光驱使用iso:FreeBSD-12.0-RELEASE-amd64-disc1.iso
* 网络使用NAT模式
* 固件类型UEFI

![虚拟机配置1](/images/freebsd/install/001.png)
![虚拟机配置2](/images/freebsd/install/002.png)

## 安装过程

### 进入安装程序

启动虚拟机，等待启动进入启动选择界面

![启动选择](/images/freebsd/install/003.png)

点击回车，或者等待超时。系统初始化之后，选择安装操作

![启动install](/images/freebsd/install/004.png)

### 选择键盘布局

这里选择默认

![选择键盘布局](/images/freebsd/install/005.png)

### 设置hostname

![设置hostname](/images/freebsd/install/006.png)

### 选择系统组件

![选择系统组件](/images/freebsd/install/007.png)

### 磁盘分区

这里选择了自动的zfs

![磁盘分区](/images/freebsd/install/008.png)

配置zfs，修改：开启磁盘加密，开启swap，开启swap加密，调整swap空间大小

![配置zfs](/images/freebsd/install/009.png)

设置zfs设备类型

![配置zfs](/images/freebsd/install/010.png)

确认警告

![确认警告](/images/freebsd/install/011.png)

![确认警告](/images/freebsd/install/012.png)

输入zfs加密密码

![输入zfs加密密码](/images/freebsd/install/013.png)
![输入zfs加密密码](/images/freebsd/install/014.png)
![确认警告](/images/freebsd/install/015.png)

等待处理

![等待处理](/images/freebsd/install/016.png)
![等待处理](/images/freebsd/install/017.png)

### 设置root用户密码

![设置root用户密码](/images/freebsd/install/018.png)

### 网络配置

选择网络设备

![选择网络设备](/images/freebsd/install/019.png)

通过DHCP设置IPV4地址

![通过DHCP设置IPV4地址](/images/freebsd/install/020.png)

通过DHCP设置IPV6地址

![通过DHCP设置IPV4地址](/images/freebsd/install/021.png)

设置DNS

![设置DNS](/images/freebsd/install/022.png)

### 设置时区

![设置时区](/images/freebsd/install/023.png)

选择亚洲 -> 中国 -> 北京时间

设置日期和时间

![设置日期](/images/freebsd/install/024.png)

![设置时间](/images/freebsd/install/025.png)

### 系统配置

选择系统启动组件

![选择系统启动组件](/images/freebsd/install/026.png)

配置系统安全组件

![配置系统安全组件](/images/freebsd/install/027.png)

### 添加普通用户

![添加普通用户](/images/freebsd/install/028.png)

![添加普通用户](/images/freebsd/install/029.png)

### 结束安装

![结束安装](/images/freebsd/install/030.png)

是否要启动一个shell 手动配置freebsd

![结束安装](/images/freebsd/install/031.png)

![结束安装](/images/freebsd/install/032.png)

