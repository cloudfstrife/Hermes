---
title: "MySQL用户及权限管理"
date: 2019-01-25T18:50:47+08:00
categories:
- Database
- MySQL
tags:
- Database
- MySQL
keywords:
- MySQL
- 用户权限管理
---

MySQL用户及权限管理

<!--more-->

## 用户管理

### 新增用户

```plaintext
create user 用户名 IDENTIFIED by '密码';
```

### 修改用户名

```plaintext
rename user feng to newuser;
```

### 修改密码

* 方法1 

```plaintext
alter user 用户名@'主机host' identified by '密码';
```
> 示例：
> 
> `alter user root@localhost identified by "root";`
> 

* 方法2

```plaintext
set password for 用户名 = '密码';
```

### 删除用户

```plaintext
drop user 用户名;
```

> MySQL 5之前删除用户时必须先使用`revoke`删除用户权限，然后删除用户
>
> MySQL 5之后`drop`命令可以删除用户的同时删除用户的相关权限 

## 权限管理

### mysql的权限

参考:[http://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html](http://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html)


| 权限 | 权限级别 | 权限说明 |
|:---|:---|:---|
| CREATE | 数据库、表或索引 | 创建数据库、表或索引权限 |
| DROP | 数据库或表 | 删除数据库或表权限 |
| GRANT OPTION | 数据库、表或保存的程序 | 赋予权限选项 |
| REFERENCES | 数据库或表 | |
| ALTER | 表 | 更改表，比如添加字段、索引等 |
| DELETE | 表 | 删除数据权限 |
| INDEX | 表 | 索引权限 |
| INSERT | 表 | 插入权限 |
| SELECT | 表 | 查询权限 |
| UPDATE | 表 | 更新权限 |
| CREATE VIEW | 视图 | 创建视图权限 |
| SHOW VIEW | 视图 | 查看视图权限 |
| ALTER ROUTINE | 存储过程 | 更改存储过程权限 |
| CREATE ROUTINE | 存储过程 | 创建存储过程权限 |
| EXECUTE | 存储过程 | 执行存储过程权限 |
| FILE | 服务器主机上的文件访问 | 文件访问权限 |
| CREATE TEMPORARY TABLES | 服务器管理 | 创建临时表权限 |
| LOCK TABLES | 服务器管理 | 锁表权限 |
| CREATE USER | 服务器管理 | 创建用户权限 |
| PROCESS | 服务器管理 | 查看进程权限 |
| RELOAD | 服务器管理 | 执行flush-hosts, flush-logs, flush-privileges, flush-status,flush-tables, flush-threads, refresh, reload等命令的权限 |
| REPLICATION CLIENT | 服务器管理 | 复制权限 |
| REPLICATION SLAVE | 服务器管理 | 复制权限 |
| SHOW DATABASES | 服务器管理 | 查看数据库权限 |
| SHUTDOWN | 服务器管理 | 关闭数据库权限 |
| SUPER | 服务器管理 | 执行kill线程权限 |


### 查看用户已有权限 

```plaintext
show grants for 用户名;
```
### 分配权限

```plaintext
GRANT 权限列表 ON 权限作用域 TO 用户名@<host> [IDENTIFIED BY '密码'] [WITH GRANT OPTION]; 
```

> **权限列表**
> 
> 参考 [mysql的权限]
> 
-------
> **作用区域**包括：
> 
> 1. \*.\*&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;整个服务器
> 1. database.\*&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;数据库的所有元素
> 1. database.table&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;数据库的特定表
> 1. GRANT Update (字段1,字段2) ON 库名.表名 TO 用户名@主机host;&ensp;&ensp;&ensp;&ensp;数据库的特定表的特定字段

---

> **host**可以取值：
>
> 1. %            		&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;匹配所有主机
> 1. localhost    		&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;localhost不会被解析成IP地址，直接通过UNIXsocket连接
> 1. xxx.xxx.xxx.xxx    &ensp;&ensp;会通过TCP/IP协议连接，并且只能在本机访问；
> 1. ::1          		&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;::1就是兼容支持ipv6的，表示同ipv4的127.0.0.1

---

> * IDENTIFIED BY 		&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;'密码' 密码是可选的，如果使用此项，将修改用户密码。 
> 
> * WITH GRANT OPTION	&ensp;&ensp;&ensp;&ensp;选项是可选，代表用户可以使用GRANT/REVOKE命令将他拥有的权限赋予其他用户。**请小心使用这项功能。**

**demo**

```plaintext
grant all privileges  on xxxx.* to username@'%';
grant super on *.* to xxxx@'%';
flush privileges;
```

> flush privileges表示刷新权限。

### 回收权限

```plaintext
REVOKE  权限列表 ON 权限作用域 FROM 用户名@<host>[,用户名@<host>];
```

各属性取值同[权限分配]

### 刷新权限

```plaintext
flush privileges;
```

新设置用户或更改密码后需用`flush privileges`刷新MySQL的系统权限相关表，否则会出现拒绝访问，还有一种方法，就是重新启动mysql服务器，来使新设置生效。 

