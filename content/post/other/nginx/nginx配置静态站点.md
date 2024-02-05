---
title: "nginx配置静态站点"
date: 2019-05-18T17:44:39+08:00
categories:
- Other
- nginx
tags:
- nginx
keywords:
- nginx
---

nginx配置静态站点与SSL

<!--more-->

```nginxconf
server {
	listen 443 ssl http2;
	server_name www.bitlogs.tech bitlogs.tech;
	ssl_certificate /etc/letsencrypt/live/bitlogs.tech/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/bitlogs.tech/privkey.pem;
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
	server_name www.bitlogs.tech bitlogs.tech;
	rewrite ^(.*) https://$server_name$1 permanent;
}

```
