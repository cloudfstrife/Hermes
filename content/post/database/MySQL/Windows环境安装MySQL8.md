---
title: "Windows环境安装MySQL8"
date: 2019-03-26T20:29:48+08:00
categories:
- Database
- MySQL
tags:
- MySQL
keywords:
- MySQL
---

Windows环境安装 MySQL 8

<!--more-->

## 下载

下载地址

[Download MySQL Community Server ](https://dev.mysql.com/downloads/mysql/)

## 解压

将下载的mysql的zip包解压到系统目录，这里以`D:\mysql-8.0.15`为MySQL根目录。

## 配置环境变量

将mysql的bin目录追加到系统环境变量PATH的后面。不同windows操作系统环境变量配置方法 

## 修改mysql配置文件

在MySQL根目录下创建my.ini文件，添加以下内容：

> 附加部分InnoDB优化项

```plaintext
[mysqld]
basedir=D:\mysql-8.0.15
datadir=D:\mysql-8.0.15\data
bind-address=0.0.0.0
port=3306

character_set_server=utf8mb4
collation-server=utf8mb4_general_ci

# innoDB优化

# innoDB缓冲池大小
# 默认 : 8M
# 建议 : 物理内存的50%~80%
innodb_buffer_pool_size=2G

# 指定MySQL几个日志组
# 默认 : 2
# 建议 : 默认值
innodb_log_files_in_group=2

# 指定在一个日志组中，每个log的大小
# 默认 : 50MB
# 上限 : 4G
# 建议 : 一般控制在几个Log文件相加大小在2G以内。
innodb_log_file_size=500M

# 日志缓冲区的大小
# 默认 : 8M
# 建议 : 默认值，具有大量事务的可以考虑设置为16M。
innodb_log_buffer_size=8M

# 控制事务的提交方式
# 这个参数只有3个值（0，1，2）.
# 0 : log buffer中的数据将以每秒一次的频率写入到log file中，且同时会进行文件系统到磁盘的同步操作，但是每个事务的commit并不会触发任何log buffer 到log file的刷新或者文件系统到磁盘的刷新操作；
# 1 : 在每次事务提交的时候将logbuffer中的数据都会写入到log file，同时也会触发文件系统到磁盘的同步；
# 2 : 事务提交会触发log buffer 到log file的刷新，但并不会触发磁盘文件系统到磁盘的同步。此外，每秒会有一次文件系统到磁盘同步操作。
# 默认 : 1
innodb_flush_log_at_trx_commit=1

# 提交1次事物刷1次，可以为n
sync_binlog=1

# 使每个Innodb的表有自已独立的表空间
# 默认 : 关闭
# 建议 : 打开
innodb_file_per_table=1 

# 限制Innodb能打开的表的数据
# 如果库里的表特别多的情况，可以适当增大为1000。
# 默认 : 300
# 建议 : 视情况决定
innodb_open_files=800 

# Innodb的IO模型
# fdatasync模式：写数据时，write这一步并不需要真正写到磁盘才算完成（可能写入到操作系统buffer中就会返回完成），真正完成是flush操作，buffer交给操作系统去flush,并且文件的元数据信息也都需要更新到磁盘。
# O_DSYNC模式：写日志操作是在write这步完成，而数据文件的写入是在flush这步通过fsync完成
# O_DIRECT模式：数据文件的写入操作是直接从mysql innodb buffer到磁盘的，并不用通过操作系统的缓冲，而真正的完成也是在flush这步,日志还是要经过OS缓冲
# 默认 : fdatasync
# Windows不用设置。linux可以选择：O_DIRECT，直接写入磁盘禁止系统Cache了
# innodb_flush_method=O_DIRECT 

# 在buffer pool缓冲中，允许Innodb的脏页的百分比，
# 默认 : 90
# 建议 : 视情况决定
innodb_max_dirty_pages_pct=30

# 读请求的后台线程数
# 默认 : 4
# 建议 : 视情况决定
innodb_read_io_threads=20

# 写请求的后台线程数
# 默认 : 4
# 建议 : 视情况决定
innodb_write_io_threads=20

# 启用碎片回收线程独立于主线程
innodb_purge_threads=1


# 慢查询时间。建议0.1~0.5 默认2，单位s。
long_query_time=0.3

# 交互等待时间和非交互等待时间
# 两参数值必须一致，且同时修改
# 默认 : 28800 (8小时)
# 建议 : 7200
interactive_timeout=7200
wait_timeout=7200

# 最大连接数，默认151
max_connections=200

init_connect='SET NAMES utf8mb4'

[mysql]
default-character-set=utf8mb4

[mysql.server]
default-character-set=utf8mb4

[mysqld_safe]
default-character-set=utf8mb4
malloc-lib=tcmalloc

[client]
default-character-set=utf8mb4
```

## 初始化数据库

```plaintext
mysqld --initialize --console
```

输出内容如下:

```plaintext
D:\mysql-8.0.15>mysqld --initialize --console
 100 200 300 400 500
 100 200 300 400 500
2019-03-26T09:40:54.253048Z 0 [System] [MY-013169] [Server] D:\mysql-8.0.15\bin\mysqld.exe (mysqld 8.0.15) initializing of server in progress as process 31376
2019-03-26T09:41:35.078775Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: xxxxxxxxxxxxxxxxxxx
2019-03-26T09:41:46.170638Z 0 [System] [MY-013170] [Server] D:\mysql-8.0.15\bin\mysqld.exe (mysqld 8.0.15) initializing of server has completed
```

> 输出中的`xxxxxxxxxxxxxxxxxxx`是一串随机字符，这是mysqld随机生成的临时root密码

## 注册服务

以**管理员身份**打开一个命令行，执行以下命令：

```plaintext
mysqld --install MySQL
```

## 启动服务

在**管理员身份**的命令行中执行以下命令：

```plaintext
net start MySQL
```

## 修改root密码

以root身份登陆

```plaintext
mysql -u root -p
```

执行命令后输入上面的临时密码

修改密码

```plaintext
alter user `root`@`%` identified by '123456';
flush privileges ;
```

退出后使用新密码验证登陆即可。

## 卸载

以**管理员身份**打开一个命令行，执行以下命令：

```plaintext
mysqld --remove MySQL
```

此处末尾的`MySQL`为注册服务时的名称。

