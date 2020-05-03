---
title: "Linux命令：useradd"
date: 2019-12-03T12:17:18+08:00
categories:
- linux
- command
tags:
- linux
- command
- useradd
keywords:
- linux
- command
- useradd
---

`useradd` 或 `adduser`命令用来建立用户帐号和创建用户的起始目录，使用权限是超级用户。

<!--more-->

## 格式

```text
useradd [-d home] [-s shell] [-c comment] [-m [-k template]] [-f inactive] [-e expire ] [-p passwd] [-r] name
```

参数说明

* **-c** ：加上备注文字，备注文字保存在passwd的备注栏中
* **-d** ：指定用户登入时的主目录，替换系统默认值 `/home/<用户名>`
* **-D** ：变更预设值
* **-e** ：指定账号的失效日期，日期格式为 `MM/DD/YY` ，例如 `01/01/19` 。缺省表示永久有效
* **-f** ：指定在密码过期后多少天即关闭该账号。如果为0账号立即被停用；如果为-1则账号一直可用。默认值为 `-1`
* **-g** ：指定用户所属的群组。值可以使组名也可以是GID。用户组必须已经存在的，期默认值为100，即 `users`
* **-G** ：指定用户所属的附加群组，每个群组使用 `,` 分隔，不可以使用空白字符
* **-m** ：自动建立用户的登入目录
* **-M** ：不要自动建立用户的登入目录
* **-n** ：取消建立以用户名称为名的群组
* **-r** ：建立系统账号
* **-s** ：指定用户登入后所使用的shell。默认值为 `/bin/bash` 
* **-u** ：指定用户ID号。该值在系统中必须是唯一的。0~499默认是保留给系统用户账号使用的，所以 **该值必须大于499** 

用户建立完成之后，使用`passwd`设定用户的密码。

示例：

```text
sudo useradd -d /home/test -m -g users -G sudo -s /bin/bash test
```
