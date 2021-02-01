---
title: "Gin添加swagger文档"
date: 2021-01-30T17:56:43+08:00
categories:
- gin
tags:
- gin
keywords:
- gin
---

本文主要介绍为 gin 框架实现的服务添加 swagger 文档支持的过程

<!--more-->

示例中实现一个简单的API，返回服务状态

代码结构

```text
├── docs                                # swag 生成的代码
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
├── main.go
├── Makefile                            # MakeFile
├── pkg
│   ├── config                          # 配置
│   │   └── config.go
│   ├── entity                          # 实体
│   │   └── healz
│   │       └── healz.go
│   ├── handler                         # gin Handler
│   │   ├── handler.go
│   │   └── healz
│   │       └── healz.go
│   └── service                         # service
│       └── healz
│           └── healz.go
├── service.yml                         # 配置文件
└── version.go                          # 版本文件，习惯性的在启动时输出一下版本，可以不要
```

## 实现Gin服务

建立基础结构

```text
$ mkdir swagger-testing
$ cd swagger-testing
$ go mod init app/swagger-testing
$ touch main.go version.go service.yml Makefile
$ mkdir -p pkg/{entity,handler,service}/healz pkg/config
$ touch pkg/{entity,handler,service}/healz/healz.go pkg/config/config.go pkg/handler/handler.go
```

### pkg/config/config.go

```go
package config

import (
	"fmt"
	"sync"

	"gopkg.in/yaml.v2"

	log "github.com/sirupsen/logrus"
	"github.com/spf13/viper"
)

// -------------------------------------------------------------------------------------

// Config config
type Config struct {
	Server Server `mapstructure:"server"`
}

// Server 服务配置
type Server struct {
	Listen string `mapstructure:"listen"`
}

func (c Config) String() string {
	if b, err := yaml.Marshal(c); err == nil {
		return string(b)
	}
	return ""
}

//Show 以 yaml 格式输出配置信息
func (c Config) Show() {
	fmt.Printf(`Config
--------------------------------------------------------------
%v--------------------------------------------------------------
`, c)
}

// -------------------------------------------------------------------------------------

// C 全局配置
var (
	C    Config
	once sync.Once
)

// Initialize 初始化
func Initialize() {
	once.Do(func() {
		v := viper.New()
		v.SetConfigName("service")
		v.AddConfigPath(".")
		v.SetConfigType("yaml")
		if err := v.ReadInConfig(); err != nil {
			log.WithError(err).Fatal("initialize config")
		}
		if err := v.Unmarshal(&C); err != nil {
			log.WithError(err).Fatal("unmarshal config")
		}
		log.Info("configuration initialized")
	})
}
```

### pkg/entity/healz/healz.go

```go
package healz

// State 服务器状态
type State struct {
	// State 服务器状态
	State string
}
```

### pkg/handler/healz/healz.go

```go
package healz

import (
    "net/http"
    
	service "app/swagger-testing/pkg/service/healz"
    
    "github.com/gin-gonic/gin"
)

// Healz 状态服务
func Healz(c *gin.Context) {
	c.JSON(http.StatusOK, service.Healz())
}
```

### pkg/service/healz/healz.go

```go
package healz

import (
	entity "app/swagger-testing/pkg/entity/healz"
)

// Healz 状态
func Healz() entity.State {
	return entity.State{
		State: "OK",
	}
}
```

### pkg/handler/handler.go

```go
package handler

import (
	"app/swagger-testing/pkg/config"
	"app/swagger-testing/pkg/handler/healz"
	"context"
	"net/http"
	"sync"
	
	log "github.com/sirupsen/logrus"
	
	"github.com/gin-gonic/gin"
)

var (
	// G Gin Engine
	G      *gin.Engine
	server *http.Server
	once   sync.Once
)

func init() {
	once.Do(func() {
		G = gin.Default()
	})
}

//Regist 注册 API
func Regist() {
	v1 := G.Group("/v1")
	v1.Handle("GET", "/healz", healz.Healz)
}

// Run 启动Http服务
func Run() {
	Regist()
	server = &http.Server{
		Addr:    config.C.Server.Listen,
		Handler: G,
	}
	if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
		log.WithError(err).Fatal("Server Run")
	}
}

// Shutdown Server
func Shutdown(ctx context.Context) error {
	return server.Shutdown(ctx)
}
```

### main.go

```go
package main

import (
	"app/swagger-testing/pkg/config"
	"app/swagger-testing/pkg/handler"
	"context"
	"os"
	"os/signal"
	"syscall"
	"time"

	log "github.com/sirupsen/logrus"
)

func init() {
	// 初始化日志
	log.SetFormatter(&log.TextFormatter{
		DisableColors: true,
		FullTimestamp: true,
	})
	log.SetLevel(log.InfoLevel)

	// 输出版本信息
	ShowVersion()
}

func main() {
	config.Initialize()
	config.C.Show()

	// 优雅关闭服务
	shutdownC := make(chan struct{})
	go func() {
		defer func() {
			shutdownC <- struct{}{}
		}()

		signalC := make(chan os.Signal)
		signal.Notify(signalC, syscall.SIGINT, syscall.SIGTERM)

		<-signalC
		log.Info("shutdown ......")
		ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
		defer cancel()

		// 停止 HTTP 服务
		if err := handler.Shutdown(ctx); err != nil {
			log.WithError(err).Fatal("shutdown server")
		}
	}()

	log.Info("CLEAR FOR TAKE OFF")

	//启动服务
	handler.Run()

	//等待关闭所有资源
	<-shutdownC
}
```

### version.go

```go
package main

import "fmt"

var (
	// Version 版本
	Version string
	// CommitHash git 提交 hash 值
	CommitHash string
	// BuildTime 构建时间
	BuildTime string
)

// ShowVersion 输出版本信息
func ShowVersion() {
	fmt.Printf(
		`Version
--------------------------------------------------------------
Version:        %s
Commit Hash:    %s
Build Time:     %s
--------------------------------------------------------------
`,
		Version, CommitHash, BuildTime,
	)
}
```

### Makefile

```makefile
# 目标可执行程序名称
NAME=swagger-testing

# 主版本
# VERSION ?= $(shell git describe --tags --always --dirty)
VERSION = v1.0.0

# git 提交 Hash
# COMMIT_HASH ?= $(shell git show -s --format=%H)
COMMIT_HASH = 

# build 时间
BUILD_TIME ?= $(shell date +%Y%m%d%H%M%S)

# go文件列表
GOFILES := $(shell find . ! -path "./vendor/*" -name "*.go")

# 支持的操作系统列表
GOOSES := linux windows darwin plan9
# 支持的CPU架构
GOARCHES := 386 amd64

# 目标输出目录
DIST_FOLDER := dist

# 版本构建目录
RELEASE_FOLDER := release

# 构建附加选项
BUILD_OPTS := -ldflags "-s -w -X 'main.Version=${VERSION}' -X 'main.CommitHash=${COMMIT_HASH}' -X 'main.BuildTime=${BUILD_TIME}'"

# 单元测试附加选项
TEST_OPTS := -v

# 基准测试附加选项
BENCHMARK_OPTS := -cpu 1,2,3,4,5,6,7,8

# sonar 相关报告输出路径（包括：单元测试报告输出，单元测试覆盖率报告，golint 报告，golangci-lint 报告）
REPORT_FOLDER := sonar

# sonar report
TEST_REPORT := ${REPORT_FOLDER}/test.report 
COVER_REPORT := ${REPORT_FOLDER}/cover.report
GOLANGCI_LINT_REPORT := ${REPORT_FOLDER}/golangci-lint.xml 
GOLINT_REPORT := ${REPORT_FOLDER}/golint.report 

# Docker 镜像仓库
REGISTRY ?= repo.xxxx.xxx/xxxx

.PHONY: build format test benchmark sonar all clean container push-container

.DEFAULT: build 

build: ${DIST_FOLDER}/${NAME}/${NAME}

# 构建目标
${DIST_FOLDER}/${NAME}/${NAME}: ${GOFILES}
	go build ${BUILD_OPTS} -o $@ 

# 格式化
format:
	@for f in ${GOFILES} ; do 																			\
		gofmt -w $${f};																					\
	done

# 单元测试
test: 
	go test ${TEST_OPTS} ./...

# 基准测试
benchmark:
	go test -bench . -run ^$$ ${BENCHMARK_OPTS}  ./...

# sonar
sonar: 
	mkdir -p ${REPORT_FOLDER}
	go test -json ./... > ${TEST_REPORT}
	go test -coverprofile=${COVER_REPORT} ./... 
	golangci-lint run --out-format checkstyle  ./... > ${GOLANGCI_LINT_REPORT}
	golint ./... > ${GOLINT_REPORT}
	sonar-scanner

# 构建所有支持的操作系统和架构的目标文件
all:
	@for os in ${GOOSES} ; do																			\
		for arch in ${GOARCHES} ; do 																	\
			if [ "$${os}" = "windows" ] ;then															\
				GOOS=$${os} GOARCH=$${arch}  															\
				go build ${BUILD_OPTS} -o ${DIST_FOLDER}/${NAME}/$${os}_$${arch}/${NAME}.exe ;			\
			else																						\
				GOOS=$${os} GOARCH=$${arch}  															\
				go build ${BUILD_OPTS} -o ${DIST_FOLDER}/${NAME}/$${os}_$${arch}/${NAME} ;				\
			fi																							\
		done																							\
	done

container: ${DIST_FOLDER}/${NAME}/${NAME}
	docker build -t ${REGISTRY}/${NAME}:${VERSION}-${BUILD_TIME}				\
	    -f ${RELEASE_FOLDER}/Dockerfile .;

push-container: container
	docker push ${REGISTRY}/${NAME}:${VERSION}-${BUILD_TIME}

# 清理
clean:
	-rm -rf $(DIST_FOLDER)/*
	-rm -f ${TEST_REPORT}
	-rm -f ${COVER_REPORT}
	-rm -f ${GOLANGCI_LINT_REPORT}
	-rm -f ${GOLINT_REPORT}
```

### service.yml

```yaml
server:
  listen: ":8080"
```

### 构建与运行

```text
$ make 
go build -ldflags "-s -w -X 'main.Version=v1.0.0' -X 'main.CommitHash=' -X 'main.BuildTime=20210130183831'" -o dist/swagger-testing/swagger-testing 
$ ./dist/swagger-testing/swagger-testing
```

另启动一个终端

```text
$ curl http://127.0.0.1:8080/v1/healz
{"State":"OK"}
```

## 添加 swagger 支持

要为 gin 项目添加 swagger 文档支持有以下步骤

1. 为 main 函数添加通用注解
2. 为API添加API注解 
3. 使用 `swag` 工具生成 `docs` 目录
4. 注册 swagger 的 gin router

### 为 main 函数添加注解

**main.go**

```go
// @title swagger testing
// @version 1.0
// @description swagger testing
// @contact.name cfs
// @contact.email cfs@gmail.com
// @host 127.0.0.1:8080
// @tag.name 通用服务
func main() {
    .......
}
```

### 为API添加API注解

**pkg/handler/healz/healz.go**

```go
// Healz 状态服务
// @Summary Health checker
// @Description Health checker
// @tags 通用服务
// @Accept json
// @Produce json
// @Success 200 {object} healz.State
// @Router /v1/healz [get]
func Healz(c *gin.Context) {
	c.JSON(http.StatusOK, service.Healz())
}
```

### 使用 swag 工具生成 docs 目录

安装 swag 工具

```text
$ go get -u github.com/swaggo/swag/cmd/swag
```

生成 docs

```text
$ swag init
```

### 注册 swagger 的 gin router

**pkg/handler/handler.go**

```go
import (
......
	_ "app/swagger-testing/docs"
	ginSwagger "github.com/swaggo/gin-swagger"
	"github.com/swaggo/gin-swagger/swaggerFiles"
......
)

......

//Regist 注册 API
func Regist() {
    // 注册swagger
	G.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))

	v1 := G.Group("/v1")
	v1.Handle("GET", "/healz", healz.Healz)
}
......
```

### 重新编译运行

```text
$ make
$ ./dist/swagger-testing/swagger-testing 
```

打开浏览器访问 http://127.0.0.1:8080/swagger/index.html 即可

### 附加说明

* 可以 Makefile 的编译命令前面添加 `swag init` ，就不需要每次构建前手动生成，也可以使用 go generate 来实现。
* gin swagger 注解 参见： [swag README](https://github.com/swaggo/swag/blob/master/README.md)
* 文件上传的注解： `// @Param file formData file true "文件"`


## 参考资料

[gin-swagger](https://github.com/swaggo/gin-swagger)

[swag README](https://github.com/swaggo/swag/blob/master/README.md)