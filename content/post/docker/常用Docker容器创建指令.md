---
title: "常用Docker容器创建指令"
date: 2019-03-20T11:26:58+08:00
categories:
- Docker
tags:
- Docker
keywords:
- tech
---

常用的Docker容器创建指令

<!--more-->

## Postgres SQL

```
docker run  -d  --name postgres \
-e POSTGRES_PASSWORD=#PG密码# \
-v /data/docker/postgres/data:/var/lib/postgresql/data \
-v /etc/localtime:/etc/localtime \
-p 5432:5432 \
postgres
```

## MySQL

```
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
mysql
```


## Redis

```
sudo docker run  -d --name redis \
-v /data/docker/redis/data:/data \
-v /etc/localtime:/etc/localtime \
-p 6379:6379 \
redis redis-server
```



## Memcached

```
sudo docker run  -d --name memcached \
-v /etc/localtime:/etc/localtime \
memcached memcached -m 64
```


## 备注

1. 如果想要让容器在docker服务启动时自动启动，在语句中加入以下内容：`--restart="always"`

---

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。