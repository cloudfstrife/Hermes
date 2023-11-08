---
title: "Linux环境下的irssi"
date: 2018-12-26T17:44:52+08:00
categories:
- Linux
tags:
- Linux
- irssi
keywords:
- Linux
- irssi
---

Linux环境下的irssi

<!--more-->

## 安装irssi

```text
sudo apt-get install irssi irssi-scripts
```

## 设置irssi自动登陆服务器并进入频道

1. 使用irssi命令启动irssi

1. 使用 `/server remove xxxx.xxxx.xxx` 命令移除所有服务器

1. 添加network

```text
/network add -nick xxxxxxxx -user xxxxxxxx -realname xxxxxxxx -sasl_username xxxxxxxx -sasl_password xxxxxxxx -sasl_mechanism PLAIN freenode
```

1. 添加服务器

```text
/server add -auto -ssl -ssl_verify -network freenode chat.freenode.net 7000
```

1. 添加自动连接频道

```text
/channel add -auto #ubuntu-cn freenode
```

## 其它命令

1. **登录：**

```text
irssi -c irc.freenode.net -p 7000 -n YOU_NICKNAME
```

2. **修改昵称：**

```text
/nick apple
```

3. **加入聊天频道：**

```text
/join #fedora，如频道需要密码，/join #fedora password
```

4. **离开单个频道：**

```text
/wc
```
5. **退出频道，不加频道名退出当前频道，后面可以跟退出原因**

```text
/part [channels] [message]
```
6. **离开一个IRC SERVER**

```text
/disconnect irc.freenode.net
```

7. **切换到相应的irc channel上查看：**

```text
Alt+1~0对应1~10的irc channel编号；

Alt+q~p对应11~20的irc channel编号；

Ctrl+n/p切换上/下一个irc channel;

PageUP/PageDn切换上/下页讯息。
```

8. **转编码：**

```text
/recode add #fedora utf8;加入此频道编码格式，/recode查看加入的编码列表。
```

9. **查看频道的所有人：**

```text
/who
```

10. **查看某人的基本资料：**

```text
/whois nickname
```

11. **给某人发私消息**

```text
/msg nickname ......
```

12. **给某人说话**

```text
/say nickname ......
```

13. **忽略某人聊天内容**

```text
/ignore <昵称>
```

14. **频道列表**

```text
/list #
```

15. **加入频道**

```text
/join #频道名称
```
16. **关于自己的信息**

```text
/me
```

17. **自动保存irc log**

```text
/SET autolog ON
```

## 服务端命令

服务端分为ChanServ(频道服务), NickServ(昵称服务) 和 MemoServ(留言服务)三类。

```text
/msg chanserv #频道服务

/msg chanserv help #获得频道服务帮助信息

/msg nickserv #昵称服务

/msg nickserv help #获得昵称服务帮助信息

/msg memoserv #留言服务

/msg memoserv help #获得留言服务帮助信息
```

### 昵称注册

```text
/msg NickServ REGISTER <name> <passwd> <email>
```

执行此命令后，邮箱会收到一封邮件，里面有一个带着确认码的命令，在irssi中执行就可以了。

### 频道注册

1. 先进入频道，看看频道是否有人注册

1. 如果没有注册，使用下面的命令注册

```text
/msg ChanServ REGISTER <channel> <passwd>
```

1.注册成功就可以管理这个频道，退出后再次登陆可以使用下面的命令重新获取权限

```text
/msg ChanServ op #channel
```

## irssi 设置忽略其它用户登陆与退出

```text
/ignore * QUITS
/ignore * JOINS
/ignore * PARTS
```

## 设置编码

```text
/recode add * utf-8
```

## freenode 中文频道 

```text
/join #go-nuts
/join #golang.tw
/join #debian-cn
/join ##Orz
```
