---
title: "Sudo保留当前用户的环境变量"
date: 2020-07-05T20:06:11+08:00
categories:
- Linux
tags:
- Linux
- sudo
keywords:
- Linux
- sudo
---

`sudo` 默认的配置 `Defaults env_reset` 会在 `sudo` 执行时，重置用户的环境变量。如果想要在 `sudo` 执行期间保留环境变量，可以配置 `sudo` 的 `env_keep`选项，以使环境变量生效。

<!--more-->

## 添加配置

```text
sudo visudo -f /etc/sudoers.d/keep_proxy
```

> 文件名只是示例，任意非隐藏文件都可以

命令会打开 nano 编辑器，在编辑器中输入

```text
Defaults env_keep += "ftp_proxy http_proxy https_proxy no_proxy"
```

> `env_keep` 的值是要保留的环境变量名的列表

保存（ `ctrl+o` ），退出（ `ctrl+x` ）
