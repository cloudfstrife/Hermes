---
title: "Docker镜像加速"
date: 2019-01-14T19:44:27+08:00
categories:
- Docker
tags:
- Docker
keywords:
- Docker
- 镜像加速
---

使用中国科技大学镜像

<!--more-->

```text
echo -e "{\n\t\"registry-mirrors\": [\"https://docker.mirrors.ustc.edu.cn/\"]\n}\n" > /etc/docker/daemon.json
sudo systemctl restart docker.service
```
