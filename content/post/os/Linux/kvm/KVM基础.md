---
title: "KVM基础"
date: 2024-03-12T13:25:58+08:00
lastmod: 2024-03-12T13:25:58+08:00

categories:
  - Linux
tags:
  - Linux
  - KVM
keywords: 
  - Linux
  - KVM

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

KVM (**K**ernel-base **V**irtual **M**achine 基于内核的虚拟机) 是一个开源的系统虚拟化模块，基于虚拟化扩展（Intel-VT 或者 AMD-V）的 X86 硬件实现开源的 Linux 原生完全虚拟化解决方案。在 KVM 中，虚拟机被实现为常规的 Linux 进程，由标准 Linux 调度程序进行调度。虚机的每个虚拟 CPU 被实现为一个常规的 Linux 进程。这使得 KMV 能够使用 Linux 内核的已有功能。

<!--more-->

KVM 是 Linux 内核中的一个模块，用户要操作 Linux 内核中的模块所提供的功能，必须在用户空间创建进程，通过系统调用的方式去操作。QEMU 就是 KVM 的管理工具。

![kvm-arch](/images/linux/kvm/kvm-arch.svg)

上面架构图上 KVM 的主要作用是提供 CPU 和内存的虚级化和客户机的 I/O 拦截。Guest OS 的部分 I/O 被 KVM 拦截后，交给QEMU处理；QEMU 运行在用户空间，提供硬件 I/O 虚拟化，通过IOCTL 系统调用访问 `/dev/kvm` 设备和 KVM 交互。

## KVM 工具集合

* libvirt: 操作和管理KVM虚机的虚拟化 API，使用 C 语言编写，可以操作 KVM , VMWare , XEN , Hyper-v , LXC 等。
* Virsh: 基于 libvirt 的 命令行工具
* Virt-Manager: 基于 libvirt 的 GUI 工具
* virt-v2v: 虚机格式迁移工具
* virt-install: 创建 KVM 虚机的命令行工具
* virt-viewer: 连接到虚机屏幕的工具
* virt-clone: 虚机克隆工具
* 其它 virt-* 工具
* sVirt：安全工具

## Debian 12 安装 KVM

### 安装

```bash
sudo apt install qemu-system libvirt-daemon-system virtinst virt-viewer virt-v2v virt-p2v 
sudo adduser $USER libvirt
sudo usermod -aG $USER libvirt-qemu
```

### 增加 `devices` 支持

编辑 `/etc/default/grub`, 主要增加 `intel_iommu=on systemd.unified_cgroup_hierarchy=0` 部分

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on systemd.unified_cgroup_hierarchy=0"
```

### 更新 grub

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg 
```

### 重启并验证

```bash
virt-host-validate
```

## 启动 GuestOS

### 下载硬盘镜像

下载地址: [https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-nocloud-amd64.qcow2](https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-nocloud-amd64.qcow2)

> 这里以 Debian 12 为例，镜像地址可能有变更，请访问： [https://www.debian.org/distrib/](https://www.debian.org/distrib/)


### 创建 GuestOS

```bash
virt-install \
--import \
--name=debian \
--vcpu=2 \
--memory=2048 \
--virt-type=kvm \
--graphics=spice \
--os-variant=debian11 \
--osinfo=debian11 \
--disk debian-12-nocloud-amd64.qcow2
```

如果不出错的话，此时会打开窗口看到启动界面。

> 这里的 `--os-variant` 和 `--osinfo` 选择 `debian11` 是因为 Debian 12 的 `virt-install` 只支持 `debian11`

> 登陆可以使用 root 用户直接登陆

> 如果不想使用云镜像，可以使用 `qemu-img create -f qcow2 -o size=20G debian.qcow2` 创建硬盘镜像，在启动的时候，再使用 `--cdrom xxxx.iso` 来手动指定初始化的安装镜像，启动后手工安装操作系统。

## 常用操作

* 列出所有 GuestOS

```bash
virsh list --all
```

* 关闭 GuestOS

```bash 
virsh shutdown --domain debian
```

* 启动 GuestOS

```
virsh start --domain debian 
```

* 连接到 GuestOS 屏幕

```bash
virt-viewer debian
```

* 删除 GuestOS

```bash
virsh undefine  --domain debian
```
