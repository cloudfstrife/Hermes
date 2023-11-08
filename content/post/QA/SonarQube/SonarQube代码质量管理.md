---
title: "SonarQube代码质量管理"
date: 2018-12-24T19:26:52+08:00
categories:
- SonarQube
tags:
- Sonar
- SonarQube
keywords:
- SonarQube
---

## Sonar简介

Sonar是一个用于代码质量管理的开源平台，用于管理源代码的质量，可以从七个维度检测代码质量，通过插件形式，可以支持二十几种编程语言（社区版14种，开发者版20种，企业版24种）的代码质量管理与检测。

* 糟糕的复杂度分布
* 重复
* 缺乏单元测试
* 没有代码标准
* 没有足够的或者过多的注释
* 潜在的bug
* 糟糕的设计

<!--more-->


## 部署过程

本示例使用Docker部署SonarQube服务端，在windows10 家庭版环境使用Sonar scanner检测代码质量。
### 安装Docker

省略，详见[Docker安装](https://docs.docker.com/install/)

### 拉取Docker镜像

```text
docker pull postgres:11.1
docker pull sonarqube:7.4-community
```

### 下载SonarQube Scanners

[SonarQube Scanners 下载地址](https://docs.sonarqube.org/display/SCAN/Analyzing+Source+Code)

本示例使用命令行方式。可以跟据项目和环境选择合适的Scanner

> SonarQube Scanner for MSBuild: Launch analysis of .Net projects  
> SonarQube Scanner for Maven: Launch analysis from Maven with minimal configuration  
> SonarQube Scanner for Gradle: Launch Gradle analysis  
> SonarQube Scanner for Ant: Launch analysis from Ant  
> SonarQube Scanner For Jenkins: Launch analysis from Jenkins  
> SonarQube Scanner: Launch analysis from the command line when none of the other analyzers is appropriate  


### 创建Docker网络

```text
sudo docker network create -d bridge docker-network
```

### 创建数据库容器

```text
docker run -d --name postgres \
--network docker-network \
-e POSTGRES_PASSWORD=postgres_2016 \
-e POSTGRES_USER=sonar \
-e POSTGRES_PASSWORD=sonar \
-v /data/docker/postgres/data:/var/lib/postgresql/data \
-v /etc/localtime:/etc/localtime \
-p 5432:5432 \
postgres:11.1
```

### 创建SonarQube容器

```text
mkdir -p /data/docker/sonarqube/data
mkdir -p /data/docker/sonarqube/logs
mkdir -p /data/docker/sonarqube/extensions

sudo chown -R 999:docker /data/docker/sonarqube/data
sudo chown -R 999:docker /data/docker/sonarqube/logs
sudo chown -R 999:docker /data/docker/sonarqube/extensions

docker run -d --name sonarqube \
--network docker-network \
-e sonar.jdbc.username=sonar \
-e sonar.jdbc.password=sonar \
-e  sonar.jdbc.url="jdbc:postgresql://postgres:5432/sonar" \
-v /data/docker/sonarqube/data:/opt/sonarqube/data \
-v /data/docker/sonarqube/logs:/opt/sonarqube/logs \
-v /data/docker/sonarqube/extensions:/opt/sonarqube/extensions \
-v /etc/localtime:/etc/localtime \
-p 9000:9000 \
sonarqube:7.4-community
```

### 安装插件

访问SonarQube主页，`http://主机地址:9000`. 使用默认管理员账号：`admin/admin`登陆，点击上方的`administration`，在打开的页面中，点击`marketplace`,在搜索框中输入`chinese Pack`点击搜索结果列表的`install`按钮。稍候会提示重启，点击重启即可。

按照上面的步骤，安装其它插件，这里安装了`SonarGo`用于检测Go语言代码质量。

### 配置SonarQube Scanner

#### 解压

将下载的SonarQube Scanner 解压到目录中

#### 修改配置

修改`conf/sonar-scanner.properties`文件，将`#sonar.host.url=http://localhost:9000`修改为`sonar.host.url=http://server主机地址:9000`

#### 配置环境变量

将`bin`目录添加到`PATH`环境变量。

### 项目配置

在项目目录下添加`sonar-project.properties`文件，内容如下 ：

```properties
# 必须在SonarQube实例中唯一
sonar.projectKey=my:project
# 这是SonarQube UI中显示的名称和版本。
sonar.projectName=project_name
sonar.projectVersion=v1.0
 
# 代码路径，取值为相对sonar-project.properties的路径
sonar.sources=.
 
# 源代码编码
sonar.sourceEncoding=UTF-8
```

### 执行分析

打开命令行，进入项目目录，执行`sonar-scanner.bat`，执行结束，即可在SonarQube web管理界面看到项目的代码质量分析结束。
