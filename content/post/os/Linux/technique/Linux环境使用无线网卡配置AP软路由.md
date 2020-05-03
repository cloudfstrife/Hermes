---
title: "Linux环境使用无线网卡配置AP软路由"
date: 2018-12-26T17:19:08+08:00
categories:
- Linux
tags:
- Linux
- AP
- 软路由
keywords:
- Linux
- AP
- 软路由
---

Linux环境使用无线网卡配置AP

<!--more-->


> **环境说明**
>
> 硬件环境：Raspberry PI 3 model B
>
> 操作系统：raspbian
> 
> 有线网卡名称：eth0
> 
> 无线网卡名称：wlan0



## 步骤说明

1. 验证网卡是否支持虚拟AP功能
2. 安装做需要的组件
3. 配置hostapd
4. 配置dnsmasq
5. 配置/etc/network/interfaces开启主机的路由转发
6. 配置IPv4转发,使其可以连接网络
7. 配置无线网卡地址
8. 启动服务


### 验证网卡是否支持虚拟AP功能

因为Raspberry PI 3 model B板载两块网卡，一块有线，一块无线，所以，将树莓派做为一个无线AP是可行的，如果是主机，可以在安装好网卡驱动之后，使用命令`iw`验证网卡是否支持虚拟AP功能：

```text
[root@localhost ~]$ sudo iw list
	......
	......
	..略..
	......
	......  
Supported interface modes:  
		* IBSS  
		* managed  
		* AP  
		* AP/VLAN
```
如果「Supported interface modes」中有「AP」，则表示可以使用网卡来模拟AP，否则就要换块无线网卡了。

### 安装做需要的组件

```text
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install hostapd hostap-utils dnsmasq
```

### 配置hostapd

修改 `/etc/hostapd/hostapd.conf`，内容如下：
	
```text
interface=wlan0
driver=nl80211
ssid=RaspberryAP
channel=6
hw_mode=g
ignore_broadcast_ssid=0
auth_algs=1
wpa=3
wpa_passphrase=123456789
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP  
```

修改 `/etc/init.d/hostapd` 将文件中 `DAEMON_CONF` 的值改成：  

```text
DAEMON_CONF=/etc/hostapd/hostapd.conf
```

### 配置dnsmasq

修改 `/etc/dnsmasq.conf`，内容如下：

```text
interface=wlan0  
listen-address=192.168.100.1  
#no-dhcp-interface=  
dhcp-range=192.168.100.50,192.168.100.150,12h  
server=DNS服务器地址
```

> 上面配置了dnsmasq 监听的接口，该接口的IP、dhcp地址的范围、租期长短、dns等。


### 配置/etc/network/interfaces开启主机的路由转发

```text
sudo echo 1 >/proc/sys/net/ipv4/ip_forward
```

> 以上配置为立即生效，但重启系统后就会失效
> 
> 可以使其重启后有效的方法是修改/etc/sysct.conf文件，在其中增加如下一行：

```text
net.ipv4.ip_forward=1
```
### 配置IPv4转发,使其可以连接网络

```text
sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

### 配置无线网卡地址

```text
sudo ifconfig wlan0 192.168.100.1 netmask 255.255.255.0  up  
```

### 启动服务

```text
sudo service hostapd start
sudo service dnsmasq start
```
