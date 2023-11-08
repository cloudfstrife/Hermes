---
title: "基于Docker容器配置gitlab-CI"
date: 2019-01-14T19:56:37+08:00
categories:
- gitlab
tags:
- Docker
- Gitlab
- CI
keywords:
- Docker
- gitlab
- CI
---

基于Docker容器，部署gitlab的CI环境

<!--more-->

## 安装 Docker 

[Debian安装Docker](/2019/01/debian安装docker/)

## 部署 Gitlab 

[使用Docker构建gitlab仓库 ](/2019/01/使用docker构建gitlab仓库/)

## 创建gitlab-runner

```text
docker run -d --name gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner
```

## 注册gitlab-runner

本例做为通用的Runner注册，如果要注册为专用Runner，把密钥换成项目的Runner注册密钥，密钥在gitlab管理页面查看。

```text
docker exec -it gitlab-runner gitlab-ci-multi-runner register
```

> 此命令会要求输入一些必要信息
> 
```text
docker exec -it gitlab-runner gitlab-ci-multi-runner register
Runtime platform                                    arch=amd64 os=linux pid=12 revision=4745a6f3 version=11.8.0
Running in system-mode.                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
此处输入gitlab的URL，可以在gitlab的管理runner页面找到
Please enter the gitlab-ci token for this runner:
此处输入token，可以在gitlab的管理runner页面找到
Please enter the gitlab-ci description for this runner:
[5b211e49f51e]: 此处输入备注
Please enter the gitlab-ci tags for this runner (comma separated):
此处输入runner的标签，使用逗号分开。可以在gitlab中配置
Registering runner... succeeded                     runner=WXa7c5-U
Please enter the executor: docker-ssh, parallels, virtualbox, docker+machine, kubernetes, docker, shell, ssh, docker-ssh+machine:
指定此runner的运行器，这里输入了docker
此步的输入将影响一下步的配置，其它选项的配置略有不同。
Please enter the default Docker image (e.g. ruby:2.1):
此处指定默认使用哪个docker镜像执行CI作业
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

## 在gitlab创建项目并添加代码

略

## 添加.gitlab-ci.yml

### 此处以最简单的go项目为例

```text
.
|-- .gitlab-ci.yml
|-- LICENSE
|-- README.md
|-- go.mod
`-- main.go
```

**.gitlab-ci.yml**

```yaml
stages:
  - deploy
deploy:
    stage: deploy
    script:
      - go build
      - ./testing
    only:
      - master
    tags:
      - golang
```

**go.mod**

```text
module app/testing

go 1.12
```

**main.go**

```go
package main

import "fmt"

func main() {
	fmt.Println("CY")
}
```

提交项目提交到Master分支，就可以在项目的CI页面看以相关的作业。



## 定期清理构建容器

```text
docker rm -v `docker ps -a | grep Exited | grep "runner.*cache.*" | awk '{print $1}'`
```
