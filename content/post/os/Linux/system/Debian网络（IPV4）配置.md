---
title: "Debian网络（IPV4）配置"
date: 2019-11-27T12:20:00+08:00
categories:
- Linux
tags:
- Linux
- Debian
- networking
keywords:
- Linux
- Debian
- networking
---

Debian网络（IPV4）配置

<!--more-->

## 查看网卡名称

```text
ls /sys/class/net/
```

或者

```text
ip addr
```

假设网卡名为`eth0`

## 配置网卡和IP地址

修改文件： `/etc/network/interfaces`

DHCP自动获取IP地址

```text
auto eth0                 # 网卡开机自动挂载，连接网络
# allow-hotplug eth0      # 允许热插拔
iface eth0 inet dhcp      # dhcp表示使用动态ip
```

静态IP地址

```text
auto eth0                              # 网卡开机自动挂载，连接网络
# allow-hotplug eth0                   # 允许热插拔
iface eth0 inet static                 # static表示使用静态IP
address xxx.xxx.xxx.xxx                # 设置IP地址
netmask xxx.xxx.xxx.xxx                # 设置子网掩码
gateway xxx.xxx.xxx.xxx                # 设置网关
dns-nameservers xxx.xxx.xxx.xxx        # 设置DNS
```

> `auto` 与 `allow-hotplug`的区别  
> 如果设置的是 `auto` ，不管你插不插网线，网卡都会启用，而且运行 `/etc/init.d/networking restart` 之后网卡能自动起来  
> 如果设置的是 `allow-hotplug` ，它会在开机时启动插网线的网卡，运行 `/etc/init.d/networking restart` 之后网卡不能自动重启

## 配置DNS服务器

修改文件： `/etc/resolv.conf`

```text
nameserver x.x.x.x                     # 首选DNS
nameserver x.x.x.x                     # 备选DNS
```

> DNS优先级： `/etc/hosts` 文件 >  `/etc/network/interfaces` 中DNS >  `/etc/resolv.conf` 中DNS

## 修改主机名

修改文件： `/etc/hostname`

写入新的主机名即可

## 重启网络服务

修改完网络地址配置，需要重启 `networking` 服务以使配置生效

```text
sudo service networking restart
```

## 临时配置网络

### 配置网卡地址

```text
sudo ifconfig eth0 xxx.xxx.xxx.xxx netmask xxx.xxx.xxx.xxx up
```

### 配置网关

```text
sudo route del default	                              # 删除旧的默认网关
sudo route add default gw xxx.xxx.xxx.xxx             # 建立新的默认网关
sudo route											  # 查看路由表
```
