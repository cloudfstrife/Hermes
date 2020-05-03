---
title: "技巧：m3u8合并生成MP4"
date: 2019-12-31T18:36:08+08:00
categories:
- other
- technique
tags:
- technique
- m3u8
- mp4
keywords:
- technique
- m3u8
- mp4
---

`.m3u8`文件是指`UTF-8`编码格式的`m3u`文件。`m3u`文件是记录了一个索引纯文本文件，播放软件并不播放它，而是根据它的索引找到对应的音视频文件的地址进行在线播放。[M3U - 维基百科，自由的百科全书](https://zh.wikipedia.org/zh-hans/M3U)

<!--more-->

本文以一个示例，说明在Linux环境下，使用`ffmpeg`，通过`m3u8`下载并合并输出成`mp4`视频的过程。

[示例视频地址](http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian.html)

## 环境说明

```text
$ cat /etc/debian_version 
10.2

$ ffmpeg -version
ffmpeg version 4.1.4-1~deb10u1 Copyright (c) 2000-2019 the FFmpeg developers
built with gcc 8 (Debian 8.3.0-6)
configuration: --prefix=/usr --extra-version='1~deb10u1' --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 --enable-gpl --disable-stripping --enable-avresample --disable-filter=resample --enable-avisynth --enable-gnutls --enable-ladspa --enable-libaom --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca --enable-libcdio --enable-libcodec2 --enable-libflite --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libgme --enable-libgsm --enable-libjack --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-libpulse --enable-librsvg --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libssh --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwavpack --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzmq --enable-libzvbi --enable-lv2 --enable-omx --enable-openal --enable-opengl --enable-sdl2 --enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-chromaprint --enable-frei0r --enable-libx264 --enable-shared
libavutil      56. 22.100 / 56. 22.100
libavcodec     58. 35.100 / 58. 35.100
libavformat    58. 20.100 / 58. 20.100
libavdevice    58.  5.100 / 58.  5.100
libavfilter     7. 40.101 /  7. 40.101
libavresample   4.  0.  0 /  4.  0.  0
libswscale      5.  3.100 /  5.  3.100
libswresample   3.  3.100 /  3.  3.100
libpostproc    55.  3.100 / 55.  3.100
```

## 获取 m3u8 文件

打开浏览器访问视频地址，按`F12`打开开发者工具，切换到`网络`标签，过滤器选择`媒体`

![001](/images/other/technique/m3u_to_mp4/001.png)

### 下载m3u8文件

```text
$ wget http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian.m3u8

$ cat youdian.m3u8 
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:9
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:8.341667,
youdian0.ts
#EXTINF:2.969633,
youdian1.ts
#EXTINF:8.341667,
youdian2.ts
#EXTINF:8.074733,
youdian3.ts
#EXTINF:8.341667,
youdian4.ts
#EXTINF:5.472133,
youdian5.ts
#EXTINF:8.341667,
youdian6.ts
#EXTINF:4.170833,
youdian7.ts
#EXTINF:8.341667,
youdian8.ts
#EXTINF:8.341667,
youdian9.ts
#EXTINF:8.341667,
youdian10.ts
#EXTINF:8.341667,
youdian11.ts
#EXTINF:8.341667,
youdian12.ts
#EXTINF:8.341667,
youdian13.ts
#EXTINF:8.341667,
youdian14.ts
#EXTINF:8.341667,
youdian15.ts
#EXTINF:8.341667,
youdian16.ts
#EXTINF:8.341667,
youdian17.ts
#EXTINF:8.341667,
youdian18.ts
#EXTINF:7.073733,
youdian19.ts
#EXT-X-ENDLIST
```

## 替换文件中的ts地址

```text
$ sed -i "s/youdian/http\:\/\/www\.scwj\.net\/media\/video\/tianyingzhang2015\/kaishu\/youdian/g" youdian.m3u8 
$ cat youdian.m3u8 
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:9
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian0.ts
#EXTINF:2.969633,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian1.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian2.ts
#EXTINF:8.074733,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian3.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian4.ts
#EXTINF:5.472133,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian5.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian6.ts
#EXTINF:4.170833,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian7.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian8.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian9.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian10.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian11.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian12.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian13.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian14.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian15.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian16.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian17.ts
#EXTINF:8.341667,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian18.ts
#EXTINF:7.073733,
http://www.scwj.net/media/video/tianyingzhang2015/kaishu/youdian19.ts
#EXT-X-ENDLIST

```

## 合并与生成

```text
$ ffmpeg -protocol_whitelist "file,http,https,tcp,tls" -i youdian.m3u8 -c copy ./02.y.mp4
```

## ts加密的处理

加密后的ts文件不能直接合并或播放，需要使用key对每个ts文件进行解密。

### 获取key

使用浏览器的开发者工具，找到加密key的地址。

### 修改m3u8文件

修改m3u8文件，增加`EXT-X-KEY`

```text
#EXT-X-KEY:METHOD=AES-128,URI="http://www.example.com/20180125/key.key"
```

### 合并与生成

合并与生成操作与上面的合并与生成操作相同
