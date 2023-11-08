---
title: "nginx开启HTTP2"
date: 2020-01-02T18:13:16+08:00
categories:
- Other
- nginx
tags:
- nginx
- http2
keywords:
- nginx
- http2
---

HTTP2 规范（[RFC7540](https://tools.ietf.org/html/rfc7540)）在2015年发布，至今（2020年1月）有近5年时间。  
HTTP2的优势很多，最大的特点有：多路复用，二进制压缩报文，服务器消息推送等。  

<!--more-->

更多HTTP2的信息信息，参见：[HTTP/2 简介  |  Web Fundamentals  |  Google Developers](https://developers.google.com/web/fundamentals/performance/http2)

`nginx` 提供了非常便捷的开启 HTTP2 支持的配置

## 前置条件

* `nginx`版本不低于`1.9.5`版本
* `openSSL`版本不低于`1.0.2`版本
* 已配置 HTTPS 支持

## 修改配置

```nginxconf
server {
	# 添加 http2
	listen 443 ssl http2;
	...
}
```

重启 `nginx` 服务，打开浏览器的开发者工具，选择 `网络` 标签，选择配置显示 `协议` 列，访问站点即可看到 `HTTP/2.0` 的协议标识。


![001](/images/other/nginx/001.png)
