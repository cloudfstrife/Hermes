---
title: "Linux的特殊权限"
date: 2020-04-12T18:09:38+08:00
categories:
- linux
tags:
- linux
- user
keywords:
- linux
- user
---

Linux系统的特殊权限用于扩展基础权限的功能,使得Linux权限更加强大灵活。

<!--more-->

Linux的特殊权限主要包括为SUID，SGID，SBIT

## SUID 

SUID 全称为`Set UID`，仅对二进制程序有效。表示在程序运行期间，执行者继承二进制程序的所有者的权限。

### 示例

`/usr/bin/passwd ` 命令用于修改密码，所有用户都可以执行这个程序。但是修改密码是要修改 `/etc/shadow` 文件的，

```text
$ ls -alh /etc/shadow
-rw-r----- 1 root shadow 940 12月  5 16:10 /etc/shadow
```

`/etc/shadow` 文件只有 `root` 用户可以修改，那么一般用户是如果修改密码的呢？

```text
$ ls -alh /usr/bin/passwd 
-rwsr-xr-x 1 root root 63K 7月  27  2018 /usr/bin/passwd
```

`/usr/local/passwd` 所有者权限位为 `rws` 而不是常规的 `rwx` 这代表这个二进制程序被赋予了SUID权限。即当一般用户执行 `passwd` 命令期间，临时继承 `/usr/local/passwd` 的所有者 `root` 用户的权限，以使二进制程序可以修改`/etc/shadow`文件。

### 设置SUID

```text
## 字母表示法
## 设置SUID
chmod u+s /path/to/file
## 删除SUID
chmod u-s /path/to/file

## 数字表示法
## 设置SUID 基础权限前面加 4
chmod 4755 /path/to/file
## 删除SUID
chmod 0755 /path/to/file
```

## SGID

SGID 全称为 `Set GID` ，当 SGID 用于二进制程序上时，在程序执行期间,执行者将临时继承此程序的所属组权限；SGID 作用于目录上时，此目录下所有用户新建的目录都自动继承此目录的用户组。

### 示例

```text
$ mkdir sgid_test
$ sudo chgrp root sgid_test
$ sudo chmod 2755 sgid_test
$ ls -lh 
总用量 4.0K
drwxr-sr-x 3 cloud root  4.0K 4月  13 14:02 sgid_test
$ mkdir sgid_test/test
$ ls -lh sgid_test
总用量 4.0K
drwxr-sr-x 2 cloud root  4.0K 4月  13 14:04 test
```

> 注意新建的 `sgid_test/test` 目录，属组为上级目录 `sgid_test` 的属组。

### 设置SGID

```text
## 字母表示法
## 设置SGID
chmod g+s /path/to/file
## 删除SGID
chmod g-s /path/to/file

## 数字表示法
## 设置SGID 基础权限前面加 2
chmod 2755 /path/to/file
## 删除SGID
chmod 0755 /path/to/file
```

## SBIT

SBIT 全称为 `Sticky Bit`，只能用于目录，主要用于控制多用户读写的目录的权限访问。当一个可被多用户读写的目录被设置为 SBIT 时，其子目录只能修改自己创建文件或者目录。

### 示例

```text
$ mkdir sbit_test
$ chown root:root sbit_test
$ chmod 1777 sbit_test
$ ls -lh
总用量 4.0K
drwxrwxrwt 2 root root 4.0K 4月  13 15:11 sbit_test
```

> 注意其它用户的操作权限位，最后一位为 `t` 表示目录被赋予了 SBIT 权限

此时，使用普通用户创建不同的文件，

User1

```text
$ mkdir sbit_test/user1
```

User2

```text
$ mkdir sbit_test/user2
```

查看目录权限

```text
$ ls -lh sbit_test
总用量 8.0K
drwxr-xr-x 2 user1 users 4.0K 4月  13 15:24 user1
drwxr-xr-x 2 user2 users 4.0K 4月  13 15:25 user2
```

使用User1 操作 User2 创建的目录

```text
$ mv user2 aaa
mv: 无法将'user2' 移动至'aaa': 不允许的操作
```

### 设置SBIT

```text
## 字母表示法
## 设置SGID
chmod o+t /path/to/file
## 删除SGID
chmod o-t /path/to/file

## 数字表示法
## 设置SGID 基础权限前面加 1
chmod 1755 /path/to/file
## 删除SGID
chmod 0755 /path/to/file
```


## 总结

Linux的特征权限控制，扩展了基础权限控制的控制能力，但是依赖不能满足复杂系统与业务场景的的访问控制，更精细的控制可以使用 ACL 来进行。
