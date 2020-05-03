---
title: "MySQL8.0身份认证插件caching-sha2-password"
date: 2020-03-09T17:42:50+08:00
categories:
- Database
- MySQL
tags:
- Database
- MySQL
- caching-sha2-password
keywords:
- Database
- MySQL
- caching-sha2-password
---

在 MySQL8.0 版本之前，MySQL使用的密码认证方式是 `mysql_native_passowrd` ，在 MySQL8.0 之后，加密的方式改为 `caching_sha2_password` ，

<!--more-->

## 认证插件 caching_sha2_password 工作原理

1. `caching_sha2_password` 将用户名和密码的哈希值作为缓存条目，用户认证时，去缓存中进行匹配，如果匹配成功则认证成功
2. 如果匹配失败，则会使用 `mysql.user` 表进行校验，如果校验通过，则生成相应的缓存
3. 如果校验失败，则认证失败，访问拒绝

## 查看用户的认证插件

```sql
SELECT user,host,plugin from mysql.user ;
```

## 创建用户时指定认证插件

```sql
CREATE USER USER_NAME@'%' IDENTIFIED WITH mysql_native_password BY 'PASSWORD';
```

## 修改用户认证插件

```sql
ALTER USER 'USER_NAME'@'%' IDENTIFIED WITH mysql_native_password BY 'PASSWORD';
FLUSH PRIVILEGES;
```
