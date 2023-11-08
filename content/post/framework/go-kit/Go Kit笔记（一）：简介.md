---
title: "Go Kit笔记（一）：简介"
date: 2020-09-13T11:38:39+08:00
categories:
- go
- go-kit
tags:
- go
- go-kit
keywords:
- go
- microservices
- go-kit
---

> Go kit是用于在Go中构建微服务（或优雅的整体）的**编程工具包**。

[Go kit](https://github.com/go-kit/kit)本身不是一个框架，而是一套微服务工具集，它可以用来解决分布式系统开发中的大多数常见问题。使开发者可以专注于业务逻辑。

<!--more-->

## 微服务架构设计关注点

* Trasport 数据传输(序列化和反序列化)
* Circuit breaker 断路器
* Rate Limiter 限流器
* Logging 日志
* Metrics 指标
* Request tracing 请求追踪
* Service Discovery 服务发现


## go-kit 组件介绍

### Endpoint（端点）

Go kit 使用了抽象的 `endpoint` 来为每一个RPC建立模型。

```
type Endpoint func(ctx context.Context, request interface{}) (response interface{}, err error)
```

其他go-kit组件全部通过装饰者模式注入

```
type Middleware func(Endpoint) Endpoint
```

### Transport（传输层）

`transport` 模块提供了将特定的序列化操作绑定到 endpoint 的 Helper 方法。

### Circuit breaker（回路断路器）

`Circuitbreaker` 模块提供了很多流行的回路断路的 endpoint 适配器。

> 回路断路器可以避免雪崩，并且提高了针对间歇性错误的弹性。

### Rate limiter（限流器）

`ratelimit`模块提供了到限流器代码包的 endpoint 适配器。

### Logging（日志）

Go kit 的 `log` 模块针对日志消息的结构化和可操作性实践提供了最好的设计。 

### Metrics（Instrumentation/指标/度量/仪表盘）

Go kit 的 `metric` 模块为服务提供了通用并健壮的接口集合，可以绑定到常用的后端服务，比如 `expvar` 、 `statsd` 、 `Prometheus` 。

### Request tracing（请求跟踪）

Go kit的 tracing 模块提供了为 endpoint 和传输的增强性的绑定功能，以捕捉关于请求的信息，并把它们发送到跟踪系统中。

### kitgen（代码生成）

`kitgen`是一个命令行工具，用于快速生成 Go kit 服务代码。