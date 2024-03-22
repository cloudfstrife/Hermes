---
title: "使用OpenSSL生成证书"
date: 2024-03-21T16:12:34+08:00
lastmod: 2024-03-21T16:12:34+08:00

categories:
  - tools
tags:
  - tools
  - OpenSSL
keywords: 
  - tools
  - OpenSSL

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

[OpenSSL](https://www.openssl.org/) 是一套开源的密码学工具包，为网络通信提供安全及数据完整性的一种安全协议。它不仅提供了加密库，还包括了命令行工具，可以用于创建证书、生成密钥、测试SSL/TLS连接等。

<!--more-->

**主要功能**

* 加密算法支持: 支持多种加密算法
* SSL/TLS协议实现: 提供 SSL v2/v3 和 TLS 协议的实现
* 证书处理: 生成和管理SSL证书

**组件**

* openssl: 多用途的命令行工具
* libcrypto: 加密算法库
* libssl: 加密模块应用库，实现了SSL及TLS


**应用场景**

* 安全通信: 用于网站、API等的加密通信
* 数据加密: 用于文件、消息等的加密

本文使用 OpenSSL 简单模拟 CA 签发证书的流程

## 概念与名词解释

* **CA**: CA就相当于一个认证机构，经过这个机构签名的证书可以被认为是可信的
* **证书**: 证书就是一种文件格式，其中包含被签名的公钥、公钥持有者的相关信息、以及 CA 对公钥的签名信息

### 常用格式

* **x509**: 证书只有公钥，不包含私钥。X.509证书主要用于识别互联网通信和计算机网络中的身份，保护数据传输安全
* **pcks#7**: 主要是用于存储签名或者加密后的数据，PKCS7可以用原始的DER格式进行存储，也可以使用PEM格式进行存储
* **pcks#12**: 包含私钥和公钥，有口令保护，PKCS#12的文件是以.p12 或者 .pfx结尾的
* **PEM**: Privacy-Enhanced Mail，隐私增强邮件。PEM格式一般包含头部，说明字段，BASE64编码的内容，尾部
* **DER**: Distinguished Encoding Rules，可分辩编码规则，一种二进制格式

### 常用文件后缀

* **.pem**: PEM格式的文件
* **.der**: DER格式的文件
* **.csr**: 证书签署请求文件，是用于向 CA 申请签名的请求文件
* **.crt**: 证书文件
* **.cer**: 证书文件
* **.key**: 一般用于存储密钥文件

## CA 生成自签名证书

CA 生成自签名证书步骤如下：

1. 生成 CA 的密钥
2. 生成证书签署请求（CSR）
3. 生成 CA 证书

```bash
mkdir -p CA
cd CA/

# 生成RSA密钥
openssl genrsa -out ca.pem 4096

# 提取公钥
openssl rsa -in ca.pem -outform PEM -pubout -out ca_public.pem

# 生成证书签署请求
openssl req -new -key ca.pem -out ca.csr -subj "/C=US/ST=New York/L=Brooklyn/O=Brooklyn Company/OU=IT/CN=brooklyn.com/emailAddress=it@brooklyn.com"

# subj 说明：
# /C=US                             国家       Country Name (2 letter code) [AU]:US
# /ST=New York                      州或省名称  State or Province Name (full name) [Some-State]:New York
# /L=Brooklyn                       地名       Locality Name (eg, city) []:Brooklyn
# /O=Brooklyn Company               公司名称    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Example Brooklyn Company
# /OU=IT                            部门名称    Organizational Unit Name (eg, section) []:Technology Division
# /CN=brooklyn.com                  备注       Common Name (e.g. server FQDN or YOUR name) []:examplebrooklyn.com
# /emailAddress=it@brooklyn.com     电子邮件    Email Address []:

# 生成证书
openssl x509 -req -days 365 -in ca.csr -signkey ca.pem -out ca.crt
```

## 用户生成证书签署请求

CA 为用户签名证书的步骤如下：

1. 用户生成自己的密钥
2. 用户生成证书签署请求（CSR）
3. 用户以秘密形式将证书签署请求提交给 CA
3. CA 为证书签名
4. CA 将签名后的证书与 CA 的证书一起分发给用户

```bash
mkdir -p server
cd server

# 生成密钥
openssl genrsa -out server_private.pem 4096

# 提取公钥
openssl rsa -in server_private.pem -outform PEM -pubout -out server_public.pem

# 生成证书签署请求
openssl req -new -key server_private.pem -out server.csr -subj "/C=US/ST=State of Tennessee/L=Memphis/O=Memphis Company/OU=IT/CN=www.memphis.com/emailAddress=it@memphis.com"
```

## CA 为用户的证书签名

```bash
# 生成 v3 扩展
cat << EOF  | tee -a v3.ext
[v3_req]
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.memphis.com
EOF

# 生成证书
openssl x509 -req -sha512 \
-days 365 \
-extfile v3.ext \
-extensions v3_req \
-CA ./CA/ca.crt \
-CAkey ./CA/ca.pem \
-CAcreateserial \
-in ./server/server.csr \
-out server.crt
```

至此用户证书已经生成。

## 测试

这里以nginx为例，将 `server/server_private.pem` 和 `server.crt` 放置到指定位置，这里以 `/data` 为例，然后配置 nginx 服务

```text
server {
	listen 443 ssl http2;
	server_name www.memphis.com memphis.com;
	ssl_certificate /data/server.crt;
	ssl_certificate_key /data/server_private.pem;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
	ssl_session_timeout 5m;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_prefer_server_ciphers on;
	
	if ($host = 'bitlogs.tech' ) {
		rewrite ^/(.*)$ https://www.bitlogs.tech/$1 permanent;
	}
	
	root /data/www; 
	index index.html;
	location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
		root /data/www; 
	}
}

server {
	listen 80;
	server_name www.memphis.com memphis.com;
	rewrite ^(.*) https://$server_name$1 permanent;
}
```

启动服务后，访问页面会报 `SEC_ERROR_UNKNOWN_ISSUER` ， 因为证书的签发者是未知的。此时可以将 `CA/ca.crt` 添加到浏览器中。