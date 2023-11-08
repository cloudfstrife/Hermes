---
title: "使用Docker构建Docker私有镜像仓库"
date: 2019-03-13T18:15:58+08:00
categories:
- Docker
tags:
- Docker
- registry
keywords:
- Docker
- registry
---

使用Docker构建Docker私有镜像仓库

<!--more-->

## 拉取镜像

```text
sudo docker pull registry:2.6.2
```

## 启动Docker容器

```text
docker run -d  --restart always --name registry-001 \
-e REGISTRY_HTTP_ADDR=5001
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
-e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
-v /data/docker/registry-001/registry/:/var/lib/registry/ \
-v /data/docker/registry-001/certs/:/certs \
-p 5001:5001 \
registry:2.6.2
```

> registry镜像默认使用5000端口，如果不更换端口，则不需要`-e REGISTRY_HTTP_ADDR=5001`参数
>
> 以上启动命令的证书是非必须的：

```text
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
-e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
```

## 使用

### tag镜像

```text
docker tag debian:9.3 127.0.0.1:5000/debian:9.3
```

### 推送到registry

```text
docker push 127.0.0.1:5000/debian:9.3
```

### 测试从registry拉取镜像

```text
docker rmi 127.0.0.1:5000/debian:9.3
docker rmi debian:9.3
docker pull 127.0.0.1:5000/debian:9.3
docker images
```
