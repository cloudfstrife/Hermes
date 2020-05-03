---
title: "Linux命令：mktemp"
date: 2019-12-29T19:44:37+08:00
categories:
- linux
- command
tags:
- linux
- command
- mktemp
keywords:
- linux
- command
- mktemp
---

`mktemp`命令用于创建安全的临时文件和临时目录

<!--more-->

## 用法

```text
mktemp --help
Usage: mktemp [OPTION]... [TEMPLATE]
Create a temporary file or directory, safely, and print its name.
TEMPLATE must contain at least 3 consecutive 'X's in last component.
If TEMPLATE is not specified, use tmp.XXXXXXXXXX, and --tmpdir is implied.
Files are created u+rw, and directories u+rwx, minus umask restrictions.

  -d, --directory     create a directory, not a file
  -u, --dry-run       do not create anything; merely print a name (unsafe)
  -q, --quiet         suppress diagnostics about file/dir-creation failure
      --suffix=SUFF   append SUFF to TEMPLATE.  SUFF must not contain slash.
                        This option is implied if TEMPLATE does not end in X.
      --tmpdir[=DIR]  interpret TEMPLATE relative to DIR.  If DIR is not
                        specified, use $TMPDIR if set, else /tmp.  With
                        this option, TEMPLATE must not be an absolute name.
                        Unlike with -t, TEMPLATE may contain slashes, but
                        mktemp creates only the final component

  -p DIR              use DIR as a prefix; implies -t [deprecated]
  -t                  interpret TEMPLATE as a single file name component,
                        relative to a directory: $TMPDIR, if set; else the
                        directory specified via -p; else /tmp [deprecated]

      --help     display this help and exit
      --version  output version information and exit

Report mktemp bugs to bug-coreutils@gnu.org
GNU coreutils home page: <http://www.gnu.org/software/coreutils/>
General help using GNU software: <http://www.gnu.org/gethelp/>
For complete documentation, run: info coreutils 'mktemp invocation'
```

## 常用用法

### 在shell中使用

```bash
#!/bin/bash
TMPFILE=$(mktemp)
echo "Our temp file is $TMPFILE"
```

运行

```text
$ ./mktmp.sh 
Our temp file is /tmp/tmp.D2PMaYfXWr
$ ls -alh /tmp/tmp.D2PMaYfXWr 
-rw------- 1 root root 0 Dec 30 09:53 /tmp/tmp.D2PMaYfXWr
```

> `mktemp`创建的文件权限：创建的文件为`u+rw`，目录为`u+rwx`。

### 常用的标识

* `-d`        &emsp;&emsp;创建临时目录而不是创建文件
* `-p`        &emsp;&emsp;指定临时文件所在的目录
* `-t`        &emsp;&emsp;指定临时文件的文件名模板，模板的末尾必须至少包含三个连续的`X`表示随机字符，建议至少使用六个`X`

#### 创建临时目录

```text
$ mktemp -d 
/tmp/tmp.nfispEVX5X
$ ls -alh /tmp/tmp.nfispEVX5X/
total 16K
drwx------   2 root root 4.0K Dec 30 10:10 .
drwxrwxrwt 192 root root  12K Dec 30 10:11 ..
```

#### 使用文件名模板

```text
$ mktemp -t myt_XXXXXX
/tmp/myt_6ISrHt
```
