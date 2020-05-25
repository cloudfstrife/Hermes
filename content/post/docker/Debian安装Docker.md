---
title: "Debian安装Docker"
date: 2019-01-14T19:34:56+08:00
categories:
- Docker
tags:
- Docker
keywords:
- Docker
---

Debian安装Docker CE

<!--more-->

## 安装

###  系统要求

* Stretch 9 (stable) / Raspbian Stretch
* Jessie 8 (LTS) / Raspbian Jessie
* Wheezy 7.7 (LTS)(需要更新内核到3.10以上)

### 删除旧的版本

```text
sudo apt-get remove docker docker-engine docker.io
```

### 安装前置的软件

```text
sudo apt-get update
####################################################################
##                           debian 8 +                           ##
####################################################################
sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
####################################################################
##                            debian 7                            ##
####################################################################
sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     python-software-properties
```

### 添加Docker官方GPG Key

```text
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
## 验证GPG Key 密钥指纹：9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
sudo apt-key fingerprint 0EBFCD88
```

#### 生成apt源

```text
####################################################################
##                         X86_64 / amd64                         ## 
####################################################################
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
#####################################################################
##                              armhf                              ## 
#####################################################################
echo "deb [arch=armhf] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
```

### 安装Docker CE

```text
sudo apt-get update
sudo apt-get install docker-ce
```

### 安装后处理

以把当前用户加入到 docker 用户组。可以让没有 root 权限的用户使用Docker

```text
sudo usermod -aG docker $USER
```

## 卸载

```text
sudo apt-get purge docker-ce
sudo rm -rf /var/lib/docker
```

## 参考链接

[https://docs.docker.com/install/linux/docker-ce/debian/](https://docs.docker.com/install/linux/docker-ce/debian/)
