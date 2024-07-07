---
title: "Journal日志服务"
date: 2024-07-07T15:39:14+08:00
lastmod: 2024-07-07T15:39:14+08:00

categories:
  - linux
tags:
  - linux
  - journal
keywords: 
  - linux
  - journal

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Linux环境下的 `systemd` 使用了 Journal 来管理服务的日志。日志文件目录位于 `/run/log/journal` 与 `/var/log/journal`。

<!--more-->

## journalctl

`journalctl` 是 Journal 日志服务的管理工具

## 常用命令

查看实时日志

```bash
sudo journalctl -f
```

查看内核日志
```bash
sudo journalctl -k
```

查看服务日志

```bash
sudo journalctl -u v2ray.service
```

查看服务指定日期范围的日志

```bash
sudo journalctl --since="2018-09-21 10:21:00" --until="2018-09-21 10:22:00" -u docker
```

查看指定级别日志

```bash
sudo journalctl -p 0..4 -u docker.service
```

> 日志级别：
> 0: emerg  
> 1: alert  
> 2: crit  
> 3: err  
> 4: warning  
> 5: notice  
> 6: info  
> 7: debug  

查看 journal 日志服务的磁盘使用量

```bash
sudo journalctl --disk-usage
```

## 清理日志

清理前准备

```bash 
sudo journalctl --flush 
sudo journalctl --rotate 
```

按日期清理

```bash
sudo journalctl --vacuum-time=1s
```

按照日志体积清理 

```bash
sudo journalctl --vacuum-size=256M
```

## 配置 journal 日志服务

journal 日志服务的配置文件位于 `/etc/systemd/journald.conf`，配置项的详细说明参见： `man journald.conf`

个人的一些修改

```bash
# 启用压缩
Compress=yes

# 限流 表示在 RateLimitIntervalSec 时间段内，每个服务最多允许产生 RateLimitBurst 条日志
RateLimitIntervalSec=30s
RateLimitBurst=10000

# 最大日志文件大小
SystemMaxUse=512M
RuntimeMaxUse=64M

# 日志文件的最大保留期限。
MaxRetentionSec=1week

# 向磁盘刷写日志文件的时间间隔，默认值是五分钟。
SyncIntervalSec=5m

# 设置记录到日志文件中的最高日志等级 "emerg"(0), "alert"(1), "crit"(2), "err"(3), "warning"(4), "notice"(5), "info"(6), "debug"(7) 
MaxLevelStore=warning
```

修改命令

```bash
sudo cp /etc/systemd/journald.conf /etc/systemd/journald.conf.backup

sudo sed -i 's/^#Compress.*/Compress=yes/g' /etc/systemd/journald.conf 
sudo sed -i 's/^#RateLimitIntervalSec.*/RateLimitIntervalSec=1min/g' /etc/systemd/journald.conf 
sudo sed -i 's/^#RateLimitBurst.*/RateLimitBurst=10000/g' /etc/systemd/journald.conf 
sudo sed -i 's/^#SystemMaxUse.*/SystemMaxUse=512M/g' /etc/systemd/journald.conf 
sudo sed -i 's/^#RuntimeMaxUse.*/RuntimeMaxUse=64M/g' /etc/systemd/journald.conf 
sudo sed -i 's/^#MaxRetentionSec.*/MaxRetentionSec=1week/g' /etc/systemd/journald.conf 
sudo sed -i 's/^#SyncIntervalSec.*/SyncIntervalSec=5m/g' /etc/systemd/journald.conf 
sudo sed -i 's/^#MaxLevelStore.*/MaxLevelStore=warning/g' /etc/systemd/journald.conf 

diff /etc/systemd/journald.conf /etc/systemd/journald.conf.backup
sudo systemctl restart systemd-journald
```
