---
title: "SSH agent forwarding"
date: 2021-12-30T10:56:56+08:00
categories:
- Linux
tags:
- Linux
- ssh
keywords:
- Linux
- ssh-agent
---

`ssh-agent` 是一个运行在后台的程序，用于管理本地的密钥。SSH agent forwarding 是一项非常有用的功能，它允许用户在远端服务器进行 SSH 密钥验证时使用本地密钥进行验证，而不是将密钥保存在远端服务器。

<!--more-->

假设这样的场景: 用户从自己的计算机 SSH 到 server1 ，然后从 server1 SSH 到 server2 ， 如果不启用 SSH agent forwarding 则需要在 server1 上保存用户的 SSH 密钥对以实现从 server1 到 server2 的 SSH 验证。

## 启用 SSH agent forwarding

### 客户端

客户端启用 SSH agent forwarding 功能的方式非常简单，修改 `/etc/ssh/ssh_config` （全局启用）或者 `~/.ssh/config` （单用户启用），

```
Host *
  ForwardAgent yes
```

以上修改针对所有 SSH 连接启用了 SSH agent forwarding ，也可以针对指定服务器启用此功能

```
Host 192.168.x.xxx
  ForwardAgent yes
```

### 服务端

在服务端，openssh-server 的默认配置启用了 SSH agent forwarding ，可以使用  `sshd -T | grep allowagentforwarding` 查看配置状态，如果设置为 `no` 则需要检查 `/etc/ssh/sshd_config` 配置文件中的配置项设置。

完成以上配置，并在本地启用 ssh-agent 即可实现 SSH agent forwarding 功能。

```
## 启动 ssh-agent
$ eval $(ssh-agent)
## 添加密钥
$ ssh-add ~/.ssh/id_rsa
## ssh 连接
$ ssh testing@192.168.x.xxx
## 其它操作
$ ……
```