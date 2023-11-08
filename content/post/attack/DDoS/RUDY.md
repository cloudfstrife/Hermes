---
title: "RUDY"
date: 2023-04-05T15:40:03+08:00
lastmod: 2023-04-05T15:40:03+08:00

categories:
  - DDoS
tags:
  - attack
  - DDoS
  - RUDY
keywords:
  - attack
  - DDoS
  - RUDY

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

RUDY - "R U Dead Yet?" 是一种拒绝服务攻击，其原理是以极其缓慢的速度提交表单数据来保持 Web 服务器的阻塞。RUDY 属于低速攻击，专注于创建一些拉长的请求，而不是用大量快速请求淹没服务器。

<!--more-->

## HTTP 慢速拒绝服务攻击

HTTP 慢速拒绝服务攻击是利用现有的 HTTP 机制，在建立了与 HTTP 服务器的连接后，尽量长时间保持该连接，达到对HTTP服务器的攻击。 其主要有三种攻击类型，分别是 Slow Headers 、 Slow Body 、 Slow Read 。分别在 HTTP 通信的各阶段以较低的速度接收或者发送数据，以消耗服务器资源。

### Slow Headers

Web应用在处理 HTTP 请求之前都要先接收完所有的 HTTP 头，因为 HTTP 头中包含 HTTP 请求的元数据以及请求相关的重要信息。Slow Headers 攻击在发起一个HTTP请求，但是 HTTP 头字段不发送结束符，服务器会一直等待头信息中结束符而导致连接始终被占用。

### Slow Body

攻击者发送 Post 报文向服务器请求提交数据，将总报文长度设置为一个很大的数值，但是在随后的数据发送中，以非常低的速度，每次只发送很小的报文，这样导致服务器端一直等待攻击者发送数据。

### Slow Read

客户端向服务器发送一个完整的 HTTP 请求给服务器端 （一般为 GET 请求），然后一直保持这个连接，以很低的速度读取 HTTP Response ，比如很长一段时间客户端不读取任何数据，直到连接快超时前才读取一个字节，长时间的连接将消耗服务器的连接和内存资源。

