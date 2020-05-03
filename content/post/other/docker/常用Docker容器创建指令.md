---
title: "常用Docker容器创建指令"
date: 2019-03-20T12:26:58+08:00
categories:
- Docker
tags:
- Docker
keywords:
- Docker
---

常用的Docker容器创建指令

<!--more-->

## Postgres SQL

```text
docker run  -d  --name postgres \
-e POSTGRES_PASSWORD=#PG密码# \
-v /data/docker/postgres/data:/var/lib/postgresql/data \
-v /etc/localtime:/etc/localtime \
-p 5432:5432 \
postgres
```

## MySQL

```text
sudo docker stop mysql
sudo docker rm mysql

sudo rm -rf /data/docker/mysql/data
sudo rm -rf /data/docker/mysql/log

sudo mkdir -p /data/docker/mysql/data
sudo mkdir -p /data/docker/mysql/log

sudo chgrp docker -R /data/docker/mysql/data
sudo chgrp docker -R /data/docker/mysql/log

sudo chmod 770 -R /data/docker/mysql/data
sudo chmod 770 -R /data/docker/mysql/log

sudo docker run  -dit --name mysql \
-e MYSQL_ROOT_PASSWORD=#MySQL ROOT用户密码#  \
-e MYSQL_DATABASE=#默认创建的数据库名# \
-e MYSQL_USER=#默认创建的用户#  \
-e MYSQL_PASSWORD=#默认创建的用户的密码#  \
-v /data/docker/mysql/data:/var/lib/mysql \
-v /data/docker/mysql/log:/var/log/mysql \
-v /etc/localtime:/etc/localtime \
-p 3306:3306 \
mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```


## Redis

```text
sudo docker run  -d --name redis \
-v /data/docker/redis/data:/data \
-v /etc/localtime:/etc/localtime \
-p 6379:6379 \
redis redis-server
```

## Memcached

```text
sudo docker run  -d --name memcached \
-v /etc/localtime:/etc/localtime \
memcached memcached -m 64
```

## RabbitMQ

```text
docker run -d --name rabbitmq \
--hostname rabbitmq \
-p "4369:4369" \
-p "5671:5671" \
-p "5672:5672" \
-p "15671:15671" \
-p "15672:15672" \
-p "25672:25672" \
-v /data/docker/rabbitmq:/var/lib/rabbitmq \
-e RABBITMQ_DEFAULT_USER="rabbitmq_user_name" \
-e RABBITMQ_DEFAULT_PASS="rabbitmq_pass_word" \
rabbitmq:3.8.2-management

```

## 备注

1. 如果想要让容器在docker服务启动时自动启动，在语句中加入以下内容：`--restart="always"`
