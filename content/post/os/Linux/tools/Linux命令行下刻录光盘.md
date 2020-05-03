---
title: "Linux命令行下刻录光盘"
date: 2019-04-18T19:08:02+08:00
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

```text
nrg2iso xxxx.nrg xxxx.iso 
```

## growisofs

```text
growisofs -dvd-compat -Z /dev/sr0=xxxxxx.iso
```

## sha256校验

```text
sha256sum xxxx.iso; dd if=/dev/dvd1 bs=2048 count=$(($(stat -c "%s" xxxx.iso) / 2048)) | sha256sum
```

如果两个SHA256校验值相同代表记录OK。
