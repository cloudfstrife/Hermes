---
title: "使用docker-compose构建etcd集群"
date: 2021-02-24T17:48:24+08:00
categories:
- etcd
tags:
- etcd
- docker-compose
keywords:
- etcd
- docker-compose
---

本文以极简的方式构建一个三个节点的 etcd 集群

<!--more-->

# 建立目录

```text
$ mkdir etcd
$ cd etcd
$ touch docker-compose.yml 
```

# docker-compose.yml 

```yaml
version: "3.8"
networks:
  etcd:
services:
  node1:
    image: quay.io/coreos/etcd:v3.4.14
    ports:
      - "12379:2379"
    volumes:
      - /etc/localtime:/etc/localtime
      - /data/docker/etcd/node1/data:/var/etcd
    command: > 
            /usr/local/bin/etcd 
            --name node1 
            --data-dir /var/etcd 
            --listen-client-urls http://0.0.0.0:2379 
            --advertise-client-urls http://0.0.0.0:2379 
            --listen-peer-urls http://0.0.0.0:2380 
            --initial-advertise-peer-urls http://node1:2380 
            --initial-cluster node1=http://node1:2380,node2=http://node2:2380,node3=http://node3:2380 
            --initial-cluster-token etcd-token 
            --initial-cluster-state new 
            --log-level info 
    networks:
      - etcd
  node2:
    image: quay.io/coreos/etcd:v3.4.14
    ports:
      - "22379:2379"
    volumes:
      - /etc/localtime:/etc/localtime
      - /data/docker/etcd/node2/data:/var/etcd
    command: > 
            /usr/local/bin/etcd 
            --name node2 
            --data-dir /var/etcd 
            --listen-client-urls http://0.0.0.0:2379 
            --advertise-client-urls http://0.0.0.0:2379 
            --listen-peer-urls http://0.0.0.0:2380 
            --initial-advertise-peer-urls http://node2:2380 
            --initial-cluster node1=http://node1:2380,node2=http://node2:2380,node3=http://node3:2380 
            --initial-cluster-token etcd-token 
            --initial-cluster-state new 
            --log-level info 
    networks:
      - etcd
  node3:
    image: quay.io/coreos/etcd:v3.4.14
    ports:
      - "32379:2379"
    volumes:
      - /etc/localtime:/etc/localtime
      - /data/docker/etcd/node3/data:/var/etcd
    command: > 
            /usr/local/bin/etcd 
            --name node3 
            --data-dir /var/etcd 
            --listen-client-urls http://0.0.0.0:2379 
            --advertise-client-urls http://0.0.0.0:2379 
            --listen-peer-urls http://0.0.0.0:2380 
            --initial-advertise-peer-urls http://node3:2380 
            --initial-cluster node1=http://node1:2380,node2=http://node2:2380,node3=http://node3:2380 
            --initial-cluster-token etcd-token 
            --initial-cluster-state new 
            --log-level info 
    networks:
      - etcd
```

# 启动

```text
$ docker-compose up -d 
```

# 查看集群状态

```text
$ docker exec -t etcd_node1_1 etcdctl --endpoints="http://node1:2380,http://node2:2380,http://node3:2380" endpoint health
$ docker exec -t etcd_node1_1 etcdctl --endpoints="http://node1:2380,http://node2:2380,http://node3:2380" member list
```
