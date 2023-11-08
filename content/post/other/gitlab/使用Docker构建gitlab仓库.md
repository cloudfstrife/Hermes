---
title: "使用Docker构建gitlab仓库"
date: 2019-01-14T20:00:26+08:00
categories:
- gitlab
tags:
- Docker
- gitlab
keywords:
- Docker
- Gitlab
---

使用Docker构建gitlab仓库

<!--more-->

## 拉取镜像

```text
sudo docker pull gitlab/gitlab-ce:11.6.3-ce.0
```

## 创建Docker容器

```text
sudo docker run -d --name gitlab \
--hostname gitlab \
-e GITLAB_OMNIBUS_CONFIG="external_url 'http://xxx.xxxx.xxx.xxx'; gitlab_rails['lfs_enabled'] = true; gitlab_rails['gitlab_shell_ssh_port'] = 1022; " \
-p 443:443 -p 80:80 -p 1022:22 \
-v /data/docker/gitlab/config:/etc/gitlab \
-v /data/docker/gitlab/logs:/var/log/gitlab \
-v /data/docker/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:11.6.3-ce.0
```
> **注意替换里面的 `http://xxx.xxxx.xxx.xxx` 为域名或者宿主机IP地址** 

访问`http://xxx.xxxx.xxx.xxx`，设置`root`用户的新密码。完成之后，使用root登陆即可。
