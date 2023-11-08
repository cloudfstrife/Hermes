---
title: "Linux命令：userdel"
date: 2019-12-09T18:30:11+08:00
categories:
- linux
- command
tags:
- linux
- command
- userdel
keywords:
- linux
- command
- userdel
---
`userdel` 命令功能很简单，就是删除用户的相关数据。
<!--more-->

## 用法

```text
用法：userdel [选项] 登录名

选项：
  -f, --force                   强制删除用户账户，即使用户仍然在登录状态。也强制删除用户的主目录和邮箱，即使其它用户也使用同一个主目录或邮箱不属于指定的用户。
  -h, --help                    显示此帮助信息并推出
  -r, --remove                  删除主目录和邮件池
  -R, --root CHROOT_DIR         chroot 到的目录
  -Z, --selinux-user            为用户删除所有的 SELinux 用户映射
```

## 示例

```text
sudo userdel -rf xxx
```

## 手工删除用户

以 `xxx` 为例

```text
$ vim /etc/passwd                ## 删除以 xxx 开头的行：修改用户信息文件，删除lamp用户行
$ vim /etc/shadow                ## 删除以 xxx 开头的行，删除shadow密码信息
$ vim /etc/shadow                ## 删除以 xxx 开头的行，删除群组信息
$ vim /etc/group                 ## 删除以 xxx 开头的行，删除群组shadow密码信息
$ rm -rf /var/spod/mail/xxx      ## 删除用户邮箱
$ rm -rf/home/xxx                ## 删除用户邮箱
```

> 需要注意的是，如果用户已使用一段时间，系统其它目录中存在用户的文件，因此，如果要从系统中彻底的删除用户，最好在使用 `userdel` 前，先通过 `find / -user 用户名` 命令查找系统中属于该用户的文件并加以处理。
