---
title: "在容器中编译Linux内核"
date: 2024-08-21T14:41:57+08:00
lastmod: 2024-08-21T14:41:57+08:00

categories:
  -
tags:
  -
keywords: 
 - 

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

在容器中编译Linux内核

<!--more-->

## 环境说明

```bash 
# 宿主机系统
$ cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

# docker 版本
$ docker --version   
Docker version 27.1.2, build d01f264
```

## 前置条件

* 安装好 VS Code
* 安装好 dev container 插件

## 下载源代码及签名文件

[https://kernel.org/](https://kernel.org/)

> 这里以LST的`6.6.47`为例，下载地址如下 
>
> 源代码: [https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.47.tar.xz](https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.47.tar.xz)
>
> 签名: [https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.47.tar.sign](https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.47.tar.sign)
>
> PS: 如果下载速度比较慢，可以从[https://mirrors.kernel.org](https://mirrors.kernel.org)/中的镜像站点找离的近的镜像站点下载源代码，但是签名文件一定要从 [https://kernel.org/](https://kernel.org/) 下载。

## 解压与验证签名

```bash
# 解压 
xz --decompress linux-6.6.47.tar.xz

# 获取 Linus Torvalds 和 Greg KH 的 GPG 公钥
gpg --locate-keys torvalds@kernel.org gregkh@kernel.org

# 验证签名
gpg --verify linux-6.6.47.tar.sign
```
> 务必确保验证签名命令的输出中包含 **Good signature from "Greg Kroah-Hartman <gregkh@kernel.org>"** 或者 **完好的签名，来自于 “Greg Kroah-Hartman <gregkh@kernel.org>”** 存在
>
> 可以忽略下面的**警告：此密钥未被受信任签名认证！** 或者 **WARNING: This key is not certified with a trusted signature!**

## 解压源代码

```bash
tar xf linux-6.6.47.tar
```

## 建立devcontainer配置

```bash
mkdir -p ./linux-6.6.47/.devcontainer/

cat << EOF | tee -a ./linux-6.6.47/.devcontainer/devcontainer.json
{
	"name": "linux-kernel-develop",
	"image": "linux-kernel-develop:develop",
	"build": {
		"dockerfile": "Dockerfile"
	},
	"runArgs": [
		"--name",
		"linux-kernel-develop"
	],
	"customizations": {
		"vscode": {
			"extensions": [
				"ms-vscode.cpptools",
				"ms-vscode.makefile-tools"
			]
		}
	},
	"postCreateCommand": "uname -a"
}
EOF

cat << EOF | tee -a ./linux-6.6.47/.devcontainer/Dockerfile
FROM debian:12.6

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

ADD os-env.sh /root/

RUN bash /root/os-env.sh
EOF

cat << EOF | tee -a ./linux-6.6.47/.devcontainer/os-env.sh
#!/bin/bash

# apt
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list.d/debian.sources
apt-get update
apt-get dist-upgrade -y

# software
apt-get install -y --no-install-recommends jq locales tree bash-completion \\
build-essential bc bison flex \\
qemu-efi qemu-system qemu-user \\
libncurses5-dev libssl-dev libelf-dev 

apt-get autoclean

# locale
locale-gen zh_CN.UTF-8
dpkg-reconfigure locales
EOF
```

## 启动VSCode构建镜像

```bash
cd linux-6.6.47
code .
```

进入VS Code后，如果插件安装正确，会在右下角弹出在容器中重新打开文件夹的提示，如果未弹出，可以在VSCode的命令面板中输入`Reopen In Container` 选择提示中的对应命令打开。如果第一次打开，VSCode会构建容器镜像，并启动容器，可能需要一些时间。

等容器连接完毕，此时可以**在 VSCode 中打开一个终端**执行以下命令

```bash
make defconfig

make -j$(nproc)
```

如果运气不错的话，在一段漫长的编译之后，会输出`bzImage`构建完成的输出

```text
Kernel: arch/x86/boot/bzImage is ready  (#1)
```

那么，祝你在内核游玩愉快。