---
title: "Linux下使用vsftpd架设FTP服务器"
date: 2019-02-24T12:58:21+08:00
categories:
- Linux
tags:
- linux
- vsftpd
- FTP
keywords:
- Linux
- vsftpd
- ftp
---

本文在CentOS release 6.3 (Final)环境下，使用vsftpd建立FTP服务器

<!--more-->

## 环境说明

### 服务端

|          |                            |
|:---      |:---                        |
| 操作系统 | CentOS release 6.3 (Final) |
| vsftpd   | vsftpd-2.2.2-11.el6.x86_64 |
| IP地址   | 192.168.1.206              |

### 客户端

|                 |                                                           |
|:---             |:---                                                       |
| 操作系统        | Microsoft Windows XP Professional 版本2002 Service Pack 3 |
| ftp客户端程序   | ftp,FlashFXP4.2.5                                         |
| IP地址          | 192.168.1.85                                              |


## 操作步骤

### 安装

#### 安装vsftpd

```text
yum install vsftpd*
```

#### 安装确认安装PAM服务相关部件

```text
yum install pam*
```

#### 安装DB4部件包

```text
yum install db4*
```

### 建立系统帐号

#### 建立vsftpd服务的宿主用户

```text
useradd vsftpd -s /sbin/nologin
```
> 默认的vsftpd的服务宿主用户是root，这是不符合安全性需要的。这里建立名为vsftpd的用户，用他来作为支持vsftpd的服务宿主用户。所以该用户没有许可他登陆系统的必要，设定他为不能登陆系统的用户。

#### 建立Vsftpd虚拟宿主用户

```text
useradd overload -s /sbin/nologin
```

> 虚拟宿主用户：虚拟宿主用户并不是系统用户，他们的总体权限其实是集中寄托在一个在系统中的某一个用户身上的，所谓Vsftpd的虚拟宿主用户，就是这样一个支持着所有虚拟用户的宿主用户。由于他支撑了FTP的所有虚拟的用户，那么他本身的权限将会影响着这些虚拟的用户，因此，处于安全性的考虑，也要非常注意对该用户的权限的控制，该用户也绝对没有登陆系统的必要，这里也设定他为不能登陆系统的用户。


### 配置vsftpd的配置文件

#### 修改vsftpd主配置文件

首先备份配置文件

```text
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
```

编辑配置文件`vsftpd.conf`

```text
vim /etc/vsftpd/vsftpd.conf
```

因为配置文件内容比较多，所以，这里就把配置帖出来，里面有添加的一些注释，可以对照修改，也可以复制，不过，因为中文注释是后面加的，不知道有没有漏掉#号的地方，如果有问题，可以把中文注释的行删除。

```text
# Example config file /etc/vsftpd/vsftpd.conf
#
# The default compiled in settings are fairly paranoid. This sample file
# loosens things up a bit, to make the ftp daemon more usable.
# Please see vsftpd.conf.5 for all compiled in defaults.
#
# READ THIS: This example file is NOT an exhaustive list of vsftpd options.
# Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's
# capabilities.
#
# Allow anonymous FTP? (Beware - allowed by default if you comment this out).
##anonymous_enable=YES
#设定不允许匿名访问
anonymous_enable=NO
#
# Uncomment this to allow local users to log in.
#设定本地用户可以访问。这里主要是针对虚拟宿主用户，如果该项设定为NO那么虚拟用户将无法访问.
local_enable=YES
#
# Uncomment this to enable any form of FTP write command.
#设定可以进行写操作.
write_enable=YES
#
# Default umask for local users is 077. You may wish to change this to 022,
# if your users expect that (022 is used by most other ftpd's)
#设定上传后文件的权限掩码。这个是网上找到的，不理解^_~
local_umask=022
#
# Uncomment this to allow the anonymous FTP user to upload files. This only
# has an effect if the above global write enable is activated. Also, you will
# obviously need to create a directory writable by the FTP user.
##anon_upload_enable=YES
#禁止匿名用户上传。
anon_upload_enable=NO
#
# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
##anon_mkdir_write_enable=YES
#禁止匿名用户建立目录。
anon_mkdir_write_enable=NO
#
# Activate directory messages - messages given to remote users when they
# go into a certain directory.
dirmessage_enable=YES
#
# The target log file can be vsftpd_log_file or xferlog_file.
# This depends on setting xferlog_std_format parameter
#设定开启日志记录功能。
xferlog_enable=YES
#
# Make sure PORT transfer connections originate from port 20 (ftp-data).
#设定端口20进行数据连接。
connect_from_port_20=YES
#
# If you want, you can arrange for uploaded anonymous files to be owned by
# a different user. Note! Using "root" for uploaded files is not
# recommended!
##chown_uploads=YES
#设定禁止上传文件更改宿主。
chown_uploads=NO
#chown_username=whoever
#
# The name of log file when xferlog_enable=YES and xferlog_std_format=YES
# WARNING - changing this filename affects /etc/logrotate.d/vsftpd.log
##xferlog_file=/var/log/xferlog
#设定日志文件保存路径#####该文件默认不存在。必须要手动touch出来，并且要使宿主用户vsftpd可以操作该文件
xferlog_file=/var/log/vsftpd.log
#
# Switches between logging into vsftpd_log_file and xferlog_file files.
# NO writes to vsftpd_log_file, YES to xferlog_file
#设定日志使用标准的记录格式。
xferlog_std_format=YES
#
# You may change the default value for timing out an idle session.
#idle_session_timeout=600
#
# You may change the default value for timing out a data connection.
#data_connection_timeout=120
#
# It is recommended that you define on your system a unique user which the
# ftp server can use as a totally isolated and unprivileged user.
#设定vsftpd服务的宿主用户为vsftpd用户。
nopriv_user=vsftpd
#
# Enable this and the server will recognise asynchronous ABOR requests. Not
# recommended for security (the code is non-trivial). Not enabling it,
# however, may confuse older FTP clients.
##async_abor_enable=YES
#设定支持异步传输功能。
async_abor_enable=YES
#
# By default the server will pretend to allow ASCII mode but in fact ignore
# the request. Turn on the below options to have the server actually do ASCII
# mangling on files when in ASCII mode.
# Beware that on some FTP servers, ASCII support allows a denial of service
# attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd
# predicted this attack and has always been safe, reporting the size of the
# raw file.
# ASCII mangling is a horrible feature of the protocol.
##ascii_upload_enable=YES
##ascii_download_enable=YES
#设定支持ASCII模式的上传和下载功能。
ascii_upload_enable=YES
ascii_download_enable=YES
#
# You may fully customise the login banner string:
#设定Vsftpd的登陆标语。
ftpd_banner=Welcome to blah FTP service.
#
# You may specify a file of disallowed anonymous e-mail addresses. Apparently
# useful for combatting certain DoS attacks.
#deny_email_enable=YES
# (default follows)
#banned_email_file=/etc/vsftpd/banned_emails
#
# You may specify an explicit list of local users to chroot() to their home
# directory. If chroot_local_user is YES, then this list becomes a list of
# users to NOT chroot().
#chroot_local_user=YES
##chroot_list_enable=YES
#禁止用户离开自己的FTP主目录。
chroot_list_enable=NO
# (default follows)
#chroot_list_file=/etc/vsftpd/chroot_list
#
# You may activate the "-R" option to the builtin ls. This is disabled by
# default to avoid remote users being able to cause excessive I/O on large
# sites. However, some broken FTP clients such as "ncftp" and "mirror" assume
# the presence of the "-R" option, so there is a strong case for enabling it.
##ls_recurse_enable=YES
#禁止用户登陆FTP后使用"ls -R"的命令，这里说明一下，ls -R会大量消耗系统资源。所以这里不给操作功能
ls_recurse_enable=NO
#
# When "listen" directive is enabled, vsftpd runs in standalone mode and
# listens on IPv4 sockets. This directive cannot be used in conjunction
# with the listen_ipv6 directive.
listen=YES
#
# This directive enables listening on IPv6 sockets. To listen on IPv4 and IPv6
# sockets, you must run two copies of vsftpd with two configuration files.
# Make sure, that one of the listen options is commented !!
#listen_ipv6=YES
#设定PAM服务下vsftpd的验证配置文件名。PAM验证将参考/etc/pam.d/下的vsftpd文件配置。
pam_service_name=vsftpd
userlist_enable=YES
#设定支持TCP Wrappers
tcp_wrappers=YES

########以下为默认配置中没有的选项。自己手动添加的##########
#设定启用虚拟用户功能。
guest_enable=YES
##指定虚拟用户的宿主用户。
guest_username=overload
##设定虚拟用户的权限符合他们的宿主用户。
virtual_use_local_privs=YES
##设定虚拟用户个人Vsftp的配置文件存放路径
user_config_dir=/etc/vsftpd/vconf
```

修改完成之后，保存退出。


### 建立日志文件

```text
touch /var/log/vsftpd.log
chown vsftpd.vsftpd /var/log/vsftpd.log
```

### 建立虚拟用户配置文件存放目录

```text
mkdir /etc/vsftpd/vconf/
```

### 建立虚拟用户数据库文件

#### 建立虚拟用户名单

```text
touch /etc/vsftpd/virtusers
```

> 这个文件只是临时保存在这里

#### 编辑虚拟用户名单文件

```text
vi /etc/vsftpd/virtusers
```

在文件中添加以下内容

```text
TestUser1
Password1
TestUser2
Password2
TestUser3
Password3
TestUser4
Password4
......
```

> 在其中加入虚拟用户的用户名和口令信息的格式很简单：“一行用户名，一行口令”。

#### 生成虚拟用户数据文件

```text
db_load -T -t hash -f /etc/vsftpd/virtusers /etc/vsftpd/virtusers.db
```

> 解释一下这个命令db_load命令的-T选项允许应用程序能够将文本文件转译载入进数据库。子选项-t，追加在在-T选项后，用来指定转译载入的数据库类型。（-t可以指定的数据类型有Btree、Hash、Queue和Recon数据库。这里，接下来需要指定的是Hash型。）
>
> 生成完之后 ，ls -alh 看下有没有virtusers.db


### 修改PAM验证文件，并指定虚拟用户数据库文件进行读取

#### 备份配置文件

```text
cp /etc/pam.d/vsftpd /etc/pam.d/vsftpd.backup
```

#### 编辑vsftpd的PAM验证配置文件

```text
vi /etc/pam.d/vsftpd
```

在#%PAM-1.0后面加上以下两行

```text
auth	sufficient	pam_userdb.so	db=/etc/vsftpd/virtusers
account	sufficient	pam_userdb.so	db=/etc/vsftpd/virtusers
```

> 第一行是验证用户名和口令，第二行是验证权限，sufficient表示一旦在这里通过了本验证，那么也就不用经过下面的验证步骤。相反，就要进行下面的验证，pam_userdb.so表示审核步骤调用pam_userdb.so这个库函数进行验证，db为调用的数据库文件

### 虚拟用户的配置

首先，选一个目录做为虚拟用户的主目录，这里选择的是/opt/vsftp

#### 建立虚拟用户主目录

```text
mkdir /opt/vsftp/
```

#### 建立个虚拟用户的主目录

```text
mkdir /opt/vsftp/TestUser1
mkdir /opt/vsftp/TestUser2
……
```

#### 修改目录权限

```text
chown -R overload.overload /opt/vsftp/
```

#### 建立虚拟用户配置文件

建立虚拟用户配置文件模板

```text
touch /etc/vsftpd/vconf/vconf.tmp
```

修改虚拟用户配置文件模板

```text
vi /etc/vsftpd/vconf/vconf.tmp
```

添加以下内容

```text
#指定虚拟用户的具体主路径。
local_root=/opt/vsftp/virtuser
#设定不允许匿名用户访问。
anonymous_enable=NO
#设定允许写操作。
write_enable=YES
#设定不允许匿名用户上传。
anon_upload_enable=NO
#设定不允许匿名用户建立目录。
anon_mkdir_write_enable=NO
#设定空闲连接超时时间。
idle_session_timeout=600
#设定单次连续传输最大时间。
data_connection_timeout=120
#设定并发客户端访问个数。
max_clients=50
#设定单个客户端的最大线程数，
max_per_ip=20
#设定该用户的最大传输速率，单位b/s。我这里给的是500M，这个值跟据不同条件来修改
local_max_rate=4194304000
```

> 以上内容的值是可以调整的，主要是对用户的信息连接以及操作，进行控制

#### 对不同的用户进行订制

复制虚拟用户模版配置文件

```text
cp /etc/vsftpd/vconf/vconf.tmp /etc/vsftpd/vconf/TestUser1
```

针对具体用户进行定制，修改订制文件

```text
vi /etc/vsftpd/vconf/TestUser1
```
主要修改主目录

```text
local_root=/opt/vsftp/TestUser1
```

保存并退出，然后重复以上操作，对每个虚拟用户进行配置

### 重启vsftpd服务

```text
service vsftpd restart
```

### 测试

在客户端cmd命令行下执行

```text
ftp 192.168.1.206
```
输入口令回车，然后输入密码。

> 输入密码的时候，命令行不会显示，直接输入回车就OK）

接着就可以测试一下上传和下载

上传命令：

```text
put 本地文件名
```

下载命令

```text
get 远程文件名
```

也可以通过FlashFXP来操作

## 遇到的问题

只遇到了一个问题，就是上传和下载的时候，报了一个上传失败

```text
553 Could not create file 
```

解决办法：

```text
getsebool -a|grep ftp
```

得到以下内容

```text
allow_ftpd_anon_write --> off
allow_ftpd_full_access --> off
allow_ftpd_use_cifs --> off
allow_ftpd_use_nfs --> off
ftp_home_dir --> off
ftpd_connect_db --> off
ftpd_use_passive_mode --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
```

因为seLinux的`allow_ftpd_full_access`和`ftp_home_dir`这两项是关闭状态。通过以下命令打开

```text
setsebool -P allow_ftpd_full_access 1 
setsebool -P ftp_home_dir 1 
```

重启服务

```text
service vsftpd restart 
```

如果你遇到了其它，请Google吧……

祝各位顺利……
