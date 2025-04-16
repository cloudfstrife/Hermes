---
title: "Debian10使用kubeadm安装Kubernetes"
date: 2020-07-25T08:56:06+08:00
categories:
- kubernetes
tags:
- kubernetes
- kubeadm
keywords:
- kubernetes
- kubeadm
---

记录 Debian 10 环境下安装 `Kubernetes` 的过程，整体过程： 前置准备 -> 安装运行时容器平台 ->  安装 `kubeadm` -> 拉取镜像 -> 初始化集群 -> 安装网络插件

> 因为 **GFW** 的存在，安装 `kubeadm` 需要FQ，**GFW,FUCK YOU!**

<!--more-->

# 环境说明

Kubernetes 要求集群中的每台设备具备以下条件：

* 一台或多台运行着下列系统的机器：
    - Ubuntu 16.04+
    - Debian 9+
    - CentOS 7
    - Red Hat Enterprise Linux (RHEL) 7
    - Fedora 25+
    - HypriotOS v1.0.1+
    - Container Linux (测试 1800.6.0 版本)
* 每台机器 2 GB 以上的 RAM ， CPU 核心 2 个以上
* 集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)

## 主机列表

|     IP地址     | 主机名  |  操作系统    |
| :------------: | :-----: | :---------: |
| 192.168.220.21 | node001 | debian 10.4 |
| 192.168.220.22 | node002 | debian 10.4 |
| 192.168.220.23 | node003 | debian 10.4 |

# 前置准备

以下操作在每台主机上执行

## 检查网卡 MAC 和 product_uuid 的唯一性

网卡MAC

```text
ip link
```

product_uuid

```text
sudo cat /sys/class/dmi/id/product_uuid
```

> 确保 MAC 和 product_uuid 不重复

## 确保 iptables 工具不使用 nftables 后端

```text
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo update-alternatives --set arptables /usr/sbin/arptables-legacy
sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

## 添加hosts

> 不确定这一步是不是必要的

```text
cat << EOF | sudo tee -a /etc/hosts

192.168.220.21 node001
192.168.220.22 node002
192.168.220.23 node003

EOF
```

## 禁用 swap 分区

参考 : [禁用交换（swap）分区的正确方式](/post/os/linux/system/systemd引导的linux禁用交换swap分区的正确方式)

# 安装Docker

在集群的每台机器上执行

```text
sudo apt-get install \
   apt-transport-https \
   ca-certificates \
   curl \
   gnupg-agent \
   software-properties-common

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo usermod -aG docker $USER

cat << EOF | sudo tee -a /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

EOF

```

# 安装 kubeadm

在集群的每台机器上执行

> 执行这些命令之前，先配置 http 和 https 代理，并配置 sudo 保留当前用户配置的代理环境变量，参见： [sudo保留当前用户的环境变量](/2020/07/sudo保留当前用户的环境变量/)

```text
export http_proxy=http://${PROXY_IP_ADDRESS}:${PROXY_PORT}
export https_proxy=$http_proxy
```

```text
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl
```

# Pull Images

## 获取镜像列表

在任意一台主机执行

```text
kubeadm config images list
```

> 这个命令会输出 kubeadm 初始化集群需要的镜像，这些镜像地址都是以 `k8s.gcr.io/` 开头的，国内拉取不下来  
> 可以使用手动拉取其它用户创建的镜像，再 `tag` 的方式获得

## Pull Images 

在所有节点执行

```text
docker pull kubesphere/kube-proxy:v1.18.6
docker pull kubesphere/pause:3.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.7

docker tag kubesphere/kube-proxy:v1.18.6 k8s.gcr.io/kube-proxy:v1.18.6
docker tag kubesphere/pause:3.2 k8s.gcr.io/pause:3.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.7 k8s.gcr.io/coredns:1.6.7

docker image remove kubesphere/kube-proxy:v1.18.6
docker image remove kubesphere/pause:3.2
docker image remove registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.7
```

在 Master 节点执行

```text
docker pull kubesphere/kube-apiserver:v1.18.6
docker pull kubesphere/kube-controller-manager:v1.18.6
docker pull kubesphere/kube-scheduler:v1.18.6
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0

docker tag kubesphere/kube-apiserver:v1.18.6 k8s.gcr.io/kube-apiserver:v1.18.6
docker tag kubesphere/kube-controller-manager:v1.18.6 k8s.gcr.io/kube-controller-manager:v1.18.6
docker tag kubesphere/kube-scheduler:v1.18.6 k8s.gcr.io/kube-scheduler:v1.18.6
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0

docker image remove kubesphere/kube-apiserver:v1.18.6
docker image remove kubesphere/kube-controller-manager:v1.18.6
docker image remove kubesphere/kube-scheduler:v1.18.6
docker image remove registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
```

# 初始化集群

在 Master 节点执行

```text
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.192.0.0/16 --v=5
```

> 如果使用 flannel 网络，`--pod-network-cidr=10.244.0.0/16` 参数是必须的

命令执行成功时，结尾的输出如下

```text
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join x.x.x.x:6443 --token dkky0c.xxxxxxxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 

```

## 配置 kubectl 

在 Master 节点执行：

```text
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> 执行完之后，需要退出重新登陆一次

## Slave加入集群

在 Slave 节点以 sudo 执行最后输出的

```text
sudo kubeadm join x.x.x.x:6443 --token dkky0c.xxxxxxxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 
```

# 安装 flannel 网络插件

在主节点执行:

```text
mkdir ~/k8s
cd ~/k8s
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

# 添加自动补全

```text
cat << EOF | tee -a ~/.bashrc 

source <(kubectl completion bash)

EOF
```