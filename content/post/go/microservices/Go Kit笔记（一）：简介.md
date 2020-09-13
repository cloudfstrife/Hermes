---
title: "Go Kit笔记（一）：简介"
date: 2020-09-13T11:38:39+08:00
categories:
- go
- microservices
tags:
- go
- microservices
- go-kit
keywords:
- go
- microservices
- go-kit
---

> Go kit是用于在Go中构建微服务（或优雅的整体）的**编程工具包**。

![go-kit]()本身不是一个框架，而是一套微服务工具集，它可以用来解决分布式系统开发中的大多数常见问题。使开发者可以专注于业务逻辑。

<!--more-->

## 微服务架构设计关注点

* Rate Limiter 限流器
* Trasport 数据传输(序列化和反序列化)
* Logging 日志
* Metrics 指标
* Circuit breaker 断路器
* Request tracing 请求追踪
* Service Discovery 服务发现
