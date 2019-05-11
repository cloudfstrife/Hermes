---
title: "Docker镜像加速"
date: 2019-01-14T19:44:27+08:00
categories:
- Docker
tags:
- Docker
keywords:
- Docker
- 镜像加速
---

使用中国科技大学镜像

<!--more-->

```
echo -e "{\n\t\"registry-mirrors\": [\"https://docker.mirrors.ustc.edu.cn/\"]\n}\n" > /etc/docker/daemon.json
sudo systemctl restart docker.service
```

---

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。