---
title: "使用minikube部署k8s集群(KVM)"
date: 2023-05-14T20:24:29+08:00
lastmod: 2023-05-14T20:24:29+08:00

categories:
  - kubernetes
tags:
  - kubernetes
  - minikube
keywords: 
  - kubernetes
  - minikube

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Linux 环境使用 minikube 部署 k8s 集群

<!--more-->

# 环境说明

* Debian GNU/Linux
* KVM

# 安装 KVM

```console
sudo apt update
sudo apt install --no-install-recommends qemu-system libvirt-clients libvirt-daemon-system
sudo adduser $USER libvirt
```

# 安装 minikube

```console
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

# 安装 kubectl

```console
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
# 安装 Helm

```console
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

# 配置

```console
minikube config set driver kvm2
```

# 启动集群

```console
minikube start \
--nodes=N \
--cpus=2 \
--memory=2g \
--cni='flannel' \
--image-mirror-country='cn' \
--image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'
```

> 节点数 `--nodes=N` 可以跟据机器配置调整
> `--cpus` 表示单个节点的 CPU 核心数
> `--memory` 表示单个节点的内存大小，可以使用单位 :  `b`, `k`, `m`, `g`

# 验证

```console
kubectl get pods --all-namespace
```