---
title: "Linux命令行下刻录光盘"
date: 2019-04-18T09:08:02+08:00
categories:
- Linux
tags:
- Linux
- nrg2iso
- growisofs
keywords:
- Linux
- nrg2iso
- growisofs
---

Linux命令行下刻录光盘

<!--more-->

## NRG镜像转为ISO镜像

```
nrg2iso xxxx.nrg xxxx.iso 
```

## growisofs

```
growisofs -dvd-compat -Z /dev/sr0=xxxxxx.iso
```

## sha256校验

```
sha256sum xxxx.iso; dd if=/dev/dvd1 bs=2048 count=$(($(stat -c "%s" xxxx.iso) / 2048)) | sha256sum
```

如果两个SHA256校验值相同代表记录OK。

---

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。