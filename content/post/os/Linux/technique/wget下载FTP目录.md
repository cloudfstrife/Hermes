---
title: "wget下载FTP目录"
date: 2019-08-10T09:40:33+08:00
categories:
- Linux
tags:
- Linux
- wget
- mirror
keywords:
- Linux
- wget
- mirror
---

Linux环境下使用`wget`同步ftp目录与本地目录

<!--more-->

```text
 wget -nH -m --ftp-user=$USERNAME --ftp-password=$PASSWORD ftp://$HOST/PATH/
```

参数解释：

* -nH		&emsp;&emsp;&emsp;&emsp;--no-host-directories					&emsp;&emsp;&emsp;&emsp;不要创建主 (host) 目录
* -m		&emsp;&emsp;&emsp;&emsp;&nbsp;&nbsp;--mirror					&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;-N -r -l inf --no-remove-listing 的缩写形式。
* -N		&emsp;&emsp;&emsp;&emsp;&nbsp;&nbsp;--timestamping				&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&nbsp;&nbsp;只获取比本地文件新的文件
* -r		&emsp;&emsp;&emsp;&emsp;&emsp;--recursive						&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;指定递归下载
* -l		&emsp;&emsp;&emsp;&emsp;&emsp;--level=数字						&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&nbsp;最大递归深度 (inf 或 0 代表无限制，即全部下载)。
* &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;--no-remove-listing			&emsp;&emsp;&emsp;&emsp;&nbsp;&nbsp;不要删除‘.listing’文件





