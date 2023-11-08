---
title: "为MySQL普通用户赋予导出权限"
date: 2023-11-02T13:49:21+08:00
lastmod: 2023-11-02T13:49:21+08:00
categories:
- Database
- MySQL
tags:
- Database
- MySQL
keywords:
- Database
- MySQL

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

为MySQL普通用户赋予导出权限

```text
GRANT SELECT ON MYSQL_DATABASE.* TO MYSQL_USERNAME@`%`;
GRANT PROCESS ON *.* TO MYSQL_USERNAME@`%`;
FLUSH PRIVILEGES;
```
<!--more-->