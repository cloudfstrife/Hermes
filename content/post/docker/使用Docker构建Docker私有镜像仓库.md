---
title: "使用Docker构建Docker私有镜像仓库"
date: 2019-03-13T10:05:58+08:00
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

```
sudo docker pull registry:2.6.2
```

## 启动Docker容器

```
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
>
```
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
-e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
```

## 使用

### tag镜像

```
docker tag debian:9.3 127.0.0.1:5000/debian:9.3
```

### 推送到registry

```
docker push 127.0.0.1:5000/debian:9.3
```

### 测试从registry拉取镜像

```
docker rmi 127.0.0.1:5000/debian:9.3
docker rmi debian:9.3
docker pull 127.0.0.1:5000/debian:9.3
docker images
```

---

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。