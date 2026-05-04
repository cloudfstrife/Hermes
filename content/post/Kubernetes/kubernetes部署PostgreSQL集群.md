---
title: "Kubernetes部署PostgreSQL集群(stackgres-operator)"
date: 2025-05-04T14:45:29+08:00
lastmod: 2025-05-04T14:45:29+08:00

categories:
  - kubernetes
tags:
  - kubernetes
  - PostgreSQL
  - TimescaleDB
keywords: 
  - kubernetes
  - PostgreSQL
  - TimescaleDB

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

使用 stackgres-operator 部署 PostgreSQL 集群，并添加 TimescaleDB 支持

<!--more-->

## 安装 Operator

```bash 
# add repo
helm repo add stackgres-charts https://stackgres.io/downloads/stackgres-k8s/stackgres/helm/
# install operator
helm install --create-namespace --namespace stackgres stackgres-operator stackgres-charts/stackgres-operator --version 1.18.6
```

## 创建 Cluster

**sg-cluster-postgres.yaml**

```yaml
apiVersion: stackgres.io/v1
kind: SGCluster
metadata:
  name: postgres
  namespace: postgres
spec:
  instances: 1
  postgres:
    version: 'latest'
    extensions:
    - name: 'timescaledb'
  pods:
    persistentVolume: 
      size: '10Gi'
      storageClass: "nfs-csi"
```

> 上面的规格与storageClass可以跟据实际情况来配置

应用资源

```bash
kubectl apply -f sg-cluster-postgres.yaml
```

## TimescaleDB 支持

获取SGCluster的配置上信息

```bash
kubectl -n postgres get sgcluster/postgres -o jsonpath="{ .spec.configurations.sgPostgresConfig }" 
```

修改配置
> postgres-18-default  是上面的返回
>
> 在 shared_preload_libraries 中增加 timescaledb 
```bash
kubectl -n postgres edit sgPgConfig/postgres-18-default 
```

重启集群

```bash
kubectl apply -f - << EOF
apiVersion: stackgres.io/v1
kind: SGDbOps
metadata:
  name: restart-cluster-postgres
  namespace: postgres
spec:
  sgCluster: postgres
  op: restart
EOF
```

等待操作完成

```bash
kubectl -n postgres delete sgdbops.stackgres.io/restart-cluster-postgres
```

连接 Postgres 实例

```bash
kubectl -n postgres exec -it postgres-0 -- psql -U postgres
```

```sql
--- 启用 timescaledb 扩展
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;

--- 创建用户
CREATE USER statistics PASSWORD 'xxxxxxxxxxxxxxxxxxxxxxxx';

--- 创建数据库
CREATE DATABASE statistics WITH OWNER = statistics ENCODING = 'UTF8' CONNECTION LIMIT = -1;

--- 切换到创建的数据库
\c statistics;

--- 授权
grant all on SCHEMA public to statistics;

--- 退出
\q
```