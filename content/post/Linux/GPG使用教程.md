---
title: "GPG使用教程"
date: 2019-01-09T19:59:15+08:00
categories:
- Linux
- subcategory
tags:
- Linux
- GPG
- Crypto
- Sign
keywords:
- tech
---

## 什么是GPG

GnuPG（GNU Privacy Guard，GPG）是一种加密软件，它是PGP加密软件的开源替代物。GnuPG依照由IETF制定的OpenPGP技术标准设计。**GnuPG是用于加密、数字签章及产生非对称匙对的软件。**GPG 兼容 PGP（Pretty Good Privacy）的功能。

<!--more-->

## 安装

一般Linux发行版都包含GPG软件包，可以通过发行版的包管理器来安装（一般发行版都会默认安装，因为这个软件太重要了）。使用下面的命令确认当前系统是否安装了GPG

```
gpg --version
```

以下是Debian发行版的输出

```
gpg (GnuPG) 2.1.18
libgcrypt 1.7.6-beta
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /home/cloud/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2

```

## gpg帮助信息

使用命令`gpg --help`查看命令帮助

```
$ gpg --help 
gpg (GnuPG) 2.1.18
libgcrypt 1.7.6-beta
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /home/cloud/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2

Syntax: gpg [options] [files]
Sign, check, encrypt or decrypt
Default operation depends on the input data

Commands:
 
 -s, --sign                  make a signature
     --clear-sign            make a clear text signature
 -b, --detach-sign           make a detached signature
 -e, --encrypt               encrypt data
 -c, --symmetric             encryption only with symmetric cipher
 -d, --decrypt               decrypt data (default)
     --verify                verify a signature
 -k, --list-keys             list keys
     --list-signatures       list keys and signatures
     --check-signatures      list and check key signatures
     --fingerprint           list keys and fingerprints
 -K, --list-secret-keys      list secret keys
     --generate-key          generate a new key pair
     --quick-generate-key    quickly generate a new key pair
     --quick-add-uid         quickly add a new user-id
     --quick-revoke-uid      quickly revoke a user-id
     --quick-set-expire      quickly set a new expiration date
     --full-generate-key     full featured key pair generation
     --generate-revocation   generate a revocation certificate
     --delete-keys           remove keys from the public keyring
     --delete-secret-keys    remove keys from the secret keyring
     --quick-sign-key        quickly sign a key
     --quick-lsign-key       quickly sign a key locally
     --sign-key              sign a key
     --lsign-key             sign a key locally
     --edit-key              sign or edit a key
     --change-passphrase     change a passphrase
     --export                export keys
     --send-keys             export keys to a keyserver
     --receive-keys          import keys from a keyserver
     --search-keys           search for keys on a keyserver
     --refresh-keys          update all keys from a keyserver
     --import                import/merge keys
     --card-status           print the card status
     --edit-card             change data on a card
     --change-pin            change a card's PIN
     --update-trustdb        update the trust database
     --print-md              print message digests
     --server                run in server mode
     --tofu-policy VALUE     set the TOFU policy for a key

Options:
 
 -a, --armor                 create ascii armored output
 -r, --recipient USER-ID     encrypt for USER-ID
 -u, --local-user USER-ID    use USER-ID to sign or decrypt
 -z N                        set compress level to N (0 disables)
     --textmode              use canonical text mode
 -o, --output FILE           write output to FILE
 -v, --verbose               verbose
 -n, --dry-run               do not make any changes
 -i, --interactive           prompt before overwriting
     --openpgp               use strict OpenPGP behavior

(See the man page for a complete listing of all commands and options)

Examples:

 -se -r Bob [file]          sign and encrypt for user Bob
 --clear-sign [file]        make a clear text signature
 --detach-sign [file]       make a detached signature
 --list-keys [names]        show keys
 --fingerprint [names]      show fingerprints

Please report bugs to <https://bugs.gnupg.org>.

```

## 常规操作

### 生成密钥

```
## 生成新的密钥对
gpg --gen-key 
gpg --generate-key 


## 以全功能形式生成新的密钥对（期间会有一些密钥的配置）
gpg --full-generate-key
gpg --full-gen-key  
```

生成的密钥一般放在`~/.gnupg`目录下

> `gpg --full-generate-key` 命令可以配置加密算法，密钥长度，过期时间，私钥的密码，等信息。
>
> 生成密钥时，会要求你做一些随机的举动（敲打键盘、移动鼠标、读写硬盘之类），以生成一个随机数。此时，可以使用`dd if=/dev/urandom of=~/a.txt bs= count=2000000`这样的操作（真的需要大量的操作）来完成随机数生成。

![生成密钥](/images/linux/gnupg/1.png)

![加密私钥](/images/linux/gnupg/2.png)

生成密钥之后 建议生成一张"撤销证书"，用于密钥作废时，可以请求外部的公钥服务器撤销公钥。

```
gpg --gen-revoke [用户ID]
```
> 此处的用户ID可以是生成密钥时的邮箱，也可以是第一行下面的ASCII串

![加密私钥](/images/linux/gnupg/4.png)

### 列出密钥

```
列出公钥
gpg --list-keys

gpg --list-key [用户ID]

列出私钥
gpg --list-secret-keys 
```

![加密私钥](/images/linux/gnupg/3.png)

### 输出密钥

公钥文件（.gnupg/pubring.gpg）是二进制形式保存的，可以将其导出成ASCII码形式的文件

```

gpg --armor --output public-key.txt --export [用户ID]
gpg --armor --output private-key.txt --export-secret-keys
```

### 分发公钥

公钥服务器是专门用于储存用户公钥的服务器。send-keys可以将公钥上传到指定的公钥服务器，供其它用户进行加解密处理。

```
gpg --send-keys [用户ID] --keyserver hkp://subkeys.pgp.net 
```
公钥服务器会使用同步机制使网络上的所有公钥服务器最终都会包含你的公钥。公钥服务器没有验证机制，所以任何人都可以上传公钥，无法保证服务器上的公钥完全可靠。一般情况下，分发公钥指纹可以让用户验证公钥是否可靠：

```
gpg --fingerprint [用户ID]
```

![加密私钥](/images/linux/gnupg/5.png)

### 输入密钥

除了生成的密钥，为了验证其它系统分发的消息或者文件，需要导入其它人分发的密钥。这时可以使用import

```
gpg --import [密钥文件]
```

示例：

```
在A主机导出密钥

gpg --armor --output public-key.txt --export xxxxxxxxxxxx@xxxxx.xx

复制到B主机

scp public-key.txt xxxx@b:~

在B主机导入
gpg --import public-key.txt 

验证指纹
gpg --fingerprint xxxxxxxxxxxx@xxxxx.xx
```

除了分发公钥文件，也可以从公钥服务器导入公钥。

```
查找公钥
gpg --keyserver hkp://subkeys.pgp.net --search-keys  [用户ID]

gpg --keyserver keyring.debian.org --recv-keys xxxxxxxx
```

示例：

导入Debian发行版的公钥参见[https://www.debian.org/CD/verify](https://www.debian.org/CD/verify)

```
gpg --keyserver keyring.debian.org --recv-keys 09EA8AC3 

```

### 加密解密文件

GPG加密操作，由发送方进行，首先获取接收方公钥，使用公钥加密文件

```
 gpg --output 加密输出文件 --recipient xxxxxxxx --encrypt 待加密文件
```

>注意参数顺序，`--encrypt 待加密文件`在最后

将加密后的文件发送到接收方，在接收方解密

```
gpg --output 解密输出文件 --decrypt 加密后的文件
```

>注意参数顺序，`--decrypt 加密后的文件`在最后
>
> 文本文件加密后，比源文件小很多。一开始以为加密失败了╮(￣▽￣)╭

### 文件签名与验签

文件签名操作由文件分发方来做，首先，分发方使用自己的密钥对文件签名，并将签名后的文件（或者文件与文件签名）分发出去。

接收者使用分发方的公钥对文件进行验签，证明文件没有被篡改。

#### 对文件签名
```
gpg --sign xxxx.xxx
```

这个命令会在当前目录下生成 xxxx.xxx.gpg，**这是签名后的文件**。采用二进制储存。

```
gpg --clear-sign xxxx.xxx
```

这个命令会生成ASCII签名后的文件。文件名为xxxx.xxx.asc。签名被添加在文件头和文件尾。

> 以上两个命令都是将签名添加到文件中，一般都不会用。

```
gpg --detach-sign xxxx.xxx
```

这个命令会在当前目录下生成 xxxx.xxx.sig **这是文件的签名**。采用二进制储存。

```
gpg --detach-sign --armor xxxx.xxx
```

这个命令会生成文件的ASCII签名xxxx.xxx.asc。

#### 验证签名


```
gpg --verify --output 输出文件 xxxx.xxx.gpg 

gpg --verify --output 输出文件 xxxx.xxx.asc

gpg --verify xxxx.xxx.sig 

gpg --verify xxxx.xxx.asc
```

上面四种验签方式对应四种签名方式。

## 参考链接

[https://zh.wikibooks.org/wiki/GPG](https://zh.wikibooks.org/wiki/GPG)