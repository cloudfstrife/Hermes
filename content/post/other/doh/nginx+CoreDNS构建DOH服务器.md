---
title: "nginx+CoreDNS构建DOH服务器"
date: 2024-02-07T14:24:58+08:00
lastmod: 2024-02-07T14:24:58+08:00

categories:
  - Other
tags:
  - DOH
  - CoreDNS
keywords: 
  - DOH
  - CoreDNS

---

nginx+CoreDNS构建DOH服务器

<!--more-->

## 安装

### 必要软件

> 不同发行版软件包包可能不同，这里以 Debian 为例

```text
sudo apt install nginx supervisor wget jq tar dnsutils
```

### CoreDNS

```bash
COREDNS_INSTALL_FOLDER=/usr/local/bin

COREDNS_VERSION=$(wget -qO- -t1 -T2 "https://api.github.com/repos/coredns/coredns/releases/latest" | jq -r '.tag_name')
COREDNS_N_VERSION=${COREDNS_VERSION:1}

# 确保获取到的版本信息正确
echo ${COREDNS_VERSION} ${COREDNS_N_VERSION} 

COREDNS_DOWNLOAD_FOLDER=$(mktemp -d)

wget https://github.com/coredns/coredns/releases/download/${COREDNS_VERSION}/coredns_${COREDNS_N_VERSION}_linux_amd64.tgz -O ${COREDNS_DOWNLOAD_FOLDER}/coredns_${COREDNS_N_VERSION}_linux_amd64.tgz
tar zxf ${COREDNS_DOWNLOAD_FOLDER}//coredns_${COREDNS_N_VERSION}_linux_amd64.tgz -C ${COREDNS_DOWNLOAD_FOLDER}
sudo mv ${COREDNS_DOWNLOAD_FOLDER}/coredns /usr/local/bin/

rm -rf ${COREDNS_DOWNLOAD_FOLDER}
unset COREDNS_DOWNLOAD_FOLDER
unset COREDNS_VERSION
unset COREDNS_N_VERSION
```

## 配置文件

### CoreDNS

```text
sudo mkdir -p /etc/CoreDNS
cat << EOF | sudo tee -a /etc/CoreDNS/Corefile
https://.:8053 {
  bind 127.0.0.1
  forward . 1.1.1.1 1.0.0.1
  log
  errors
  cache
}
EOF
```

### nginx 

> 替换 `doh.xxxx.xxx`

```text
cat << EOF | sudo tee -a /etc/nginx/conf.d/xxx.xxxx.xxx.conf
server {
  listen 443 ssl http2 fastopen=256 reuseport;
  listen [::]:443 ssl http2 fastopen=256 reuseport;
  server_name xxx.xxxx.xxx;
  ssl_certificate /etc/letsencrypt/live/xxx.xxxx.xxx/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/xxx.xxxx.xxx/privkey.pem;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;

  location /dns-query {
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Host \$http_host;
    proxy_set_header X-Real-IP \$remote_addr;
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    proxy_pass http://127.0.0.1:8053;
  }
}
EOF
```

### 生成 supervisor 配置文件

```text
sudo mkdir -p /var/log/coredns/
cat << EOF | sudo tee -a /etc/supervisor/conf.d/CoreDNS.conf
[program:coredns]
command=/usr/local/bin/coredns -conf /etc/CoreDNS/Corefile
process_name=%(program_name)s
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/var/log/coredns/stdout.log
EOF
```

## 测试

```text
dig @xxx.xxxx.xxx +https google.com 
```