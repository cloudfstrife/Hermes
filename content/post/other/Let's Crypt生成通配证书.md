---
title: "Let's Crypt生成通配域名证书"
date: 2019-05-18T18:18:54+08:00
categories:
- Other
- "Let's Crypt"
tags:
- https
- "Let's Crypt"
- 通配域名证书
keywords:
- https
- "Let's Crypt"
- 通配域名证书
---

Let's Crypt生成通配域名证书

<!--more-->

## 运行环境

| 项目      | 说明     |
| --------- | -------- |
| 操作系统  | Debian 9 |

## 安装certbot

```text
sudo apt-get update
sudo apt-get install certbot
```

## 生成域名证书


```text
sudo certbot certonly  \
-d *.bitlogs.tech \
-d bitlogs.tech \
--manual \
--preferred-challenges dns \
--server https://acme-v02.api.letsencrypt.org/directory
```

> 解释： `certonly`：只获取或更新证书，`-d xxx.xxxxx.xxx`：指定对应的域名，这里指定了通配域和顶域，`--manual`：使用交互式方式获取证书，`--preferred-challenges dns`：使用 DNS 方式校验域名所有权，`--server`：Let’s Encrypt ACME服务器地址

命令执行后，进入交互模式

```text
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel):
```

此处输入电子邮件地址

```text
-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A
```

同意协议

```text
-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: Y
```

是否允许EFF向邮箱发邮件

```text
-------------------------------------------------------------------------------
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
-------------------------------------------------------------------------------
(Y)es/(N)o: Y
```

是否确认本机IP公开做为此证书的服务器

```text
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.xxxxx.com with the following value:

kFsQHSwO1LDIeyIs7VQ66sTsYioISnfzIJU0bXgo-Zg

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue

```

到此处时，不要关键回车。因为要通过DNS验证，所以要去域名供应商的管理页面配置一条txt类型的DNS解析记录，配置完成后，稍等几分钟，确认DNS配置传播完成后再按。

![](/images/other/letscrypt/01.png)

确认的命令:

```text
nslookup -q=txt _acme-challenge.xxxx.com
```

完成

```text
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/xxxx.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/xxxx.com/privkey.pem
   Your cert will expire on 20xx-xx-xx. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

此时证书已经生成，在应用服务器上配置相应路径即可。
