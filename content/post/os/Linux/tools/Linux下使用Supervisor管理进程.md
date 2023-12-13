---
title: "Linux下使用Supervisor管理进程"
date: 2023-12-12T14:20:16+08:00
lastmod: 2023-12-12T14:20:16+08:00

categories:
  - Linux
tags:
  - Supervisor
keywords: 
 - Supervisor

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Supervisor 是一个客户端/服务端系统，允许用户在类 Unix 系统上控制多个进程。使用 Supervisor 可以很方便的监听、启动、停止、重启多个进程，当被管理的进程意外退出时，Supervisor监听到进程异常退出时，会自动重新启动。

<!--more-->

## 安装

Debian

```bash
sudo apt install supervisor
```

pip

```bash
pip install supervisor
```

## 使用

### 主要可执行程序

Supervisor 主要由三个可执行程序构成

* `supervisortd`                            Supervisor 守护进程的可执行程序
* `supervisorctl`                           Supervisor 的客户端应用程序
* `echo_supervisord_conf`                   回显配置信息，一般用于生成初始配置文件 

### 配置文件

Supervisor 的主要配置文件有以下几个

* `/lib/systemd/system/supervisor.service`
systemd 配置文件
* `/etc/supervisor/supervisord.conf`
supervisord 的配置文件
* `/etc/supervisor/conf.d/*.conf`
非必须，但是为了方便管理，一般把受控进程的配置放在这里。

## 一个示例

### 编写受控进程配置文件

假定 `testing` 可执行程序放在 `/data/deploy/testing` 目录下


```bash
cat << EOF | sudo tee -a /etc/supervisor/conf.d/testing.conf
[program:testing]
directory=/data/deploy/testing
command=/data/deploy/testing/testing -p 80%(process_num)02d
numprocs=3
process_name=%(program_name)s-%(process_num)02d
autostart=true
autorestart=true
startretries=5
redirect_stderr=true
stdout_logfile=/data/deploy/testing/output-%(process_num)02d.log
EOF
```

### 重新加载配置文件

```text
# 读取配置
sudo supervisorctl reread
# 添加受控进程
sudo supervisorctl add testing
```

### 常用命令
```text
//  启动受控进程
sudo supervisorctl start testing
//  停止受控进程
sudo supervisorctl stop testing
//  重启受控进程
sudo supervisorctl restart testing
//  查看受控进程状态
sudo supervisorctl status testing
//  查看日志 `tail [-f] <name> [stdout|stderr] (default stdout)`
sudo supervisorctl tail testing
//  查看PID
sudo supervisorctl pid testing
//  发送信号
sudo supervisorctl signal TERM testing
```
## 配置文件格式

参见： [Configuration File](http://supervisord.org/configuration.html)

### `[program:x]` 段说明

配置文件必须包含一个或多个 `program` 段以便 `supervisord` 知道如何控制进程。段头是复合值。它是单词 `program`，后面直接跟冒号，然后是程序名称。例如：头值为 `[program:foo]` 的程序名称为 `foo`。该名称在 `supervisorctl` 应用程序中使用。

#### `[program:x]` 段属性

##### directory

非必要参数，无默认值。可取值为指向目录的文件路径。`supervisord` 在启动子程序前应暂时 `chdir` 到该目录。

##### command
必要属性，无默认值。启动进程的应用程序，此应用程序可以是绝对路径，也可以是相对命令路径。程序可以接受参数，命令行可以使用双引号将包含空格的参数分组传递给程序。命令的值可以包含 Python 字符串表达式 

> Python 字符串表达式可以是 `group_name`, `host_node_name`, `program_name`, `process_num`, `numprocs`, `here`(supervisord配置文件的目录)，以及所有以 `ENV_` 开头的 `supervisord` 环境变量。 (`/path/to/programname --port=80%(process_num)02d`在运行时会展开成`/path/to/programname--port=8000`)

> 受控程序本身不应是守护进程，因为 supervisord 假定自己负责受控进程的管理（参见 [Nondaemonizing of Subprocesses](http://supervisord.org/subprocess.html#nondaemonizing-of-subprocesses) ）。

##### process_name

非必要属性，默认值为`%(program_name)s` ，进程表达式，一般不需要指定此属性，除非修改了 `numprocs`。

假设一个配置如下

```text
[program:foo]
process_name=%(program_name)s_%(process_num)02d
numprocs=3
```
则那么 `foo` 组将包含三个进程，分别名为 `foo_00`、`foo_01` 和 `foo_02`
所有日志文件名、所有环境字符串和程序命令也可以包含类似的 Python 字符串表达式，以便向每个进程传递略有不同的参数。

##### numprocs

非必要属性，默认值为`1` ，Supervisor 将按照 `numprocs` 的名称启动尽可能多的程序实例。 

numprocs_start

非必要属性，默认值为`0` ，代表 `process_num` 的起始偏移量

##### umask

非必要属性，默认值为未指定，值为八进制值(示例：002, 022)，表示进程 `umask` 值。

##### priority

非必要参数，默认值为 `999` ，程序在启动和关闭顺序中的相对优先级，用于在 `start all`和 `stop all` 时，决定执行顺序。值越小，启动时越优先，关闭时越滞后

##### user

非必要参数，代表 `supervisord` 将以此 UNIX 用户账户作为进程的账户启动受控进程。只有在 `supervisord` 以 `root` 用户身份运行时，才能切换用户

##### autostart

非必要参数，默认值为 `true` ，如果为 `true`，则此程序将在 `supervisord` 启动时自动启动。

##### startsecs

非必要参数，默认值为 `1` ，代表程序启动后需要多少秒才能启动成功(将进程从`STARTING`状态转入 `RUNNING` 状态)。如果进程启动后，小于此时间就退出了(即使进程以期望的状态(参见 `exitcodes` 属性)退出)，也会被视为启动失败。

##### startretries

非必要参数，默认值为 `3` ，代表启动重试次数，如果重试次数达到此值之后，依然无法成功，则将状态设置为 `FATAL`，在到达此值之前启动失败，则状态为 `BACKOFF`，每次重试所需的时间也会越来越长。

##### autorestart

非必要参数，默认值为 `unexpected` ，指定当进程从 `RUNNING` 状态退出时，是否自动重启。  
可取值为 `false`, `unexpected`, `true` 。如果为 `false`，则不重新启动，如果为 `true`，则不无论以什么状态退出，都重新启动。如果为 `unexpected` 则在进程的退出代码与 `exitcodes` 不相符时重新启动。

##### exitcodes

非必要参数，在 4.0 之前的版本中，默认值为 `0,2`。在 4.0 中，默认值改为 `0`。此参数与 autorestart 联用。用于决定是否重启进程。

##### stopsignal

非必要参数，默认值为 `TERM` ，用于在 stop 时向进程发送的信号。值可以是信号名称或编号。通常是下列信号之一 `TERM`、`HUP`、`INT`、`QUIT`、`KILL`、`USR1`、`USR2`

##### stopwaitsecs

非必要参数，默认值为 `10` ， 程序收到停止信号后，等待操作系统向 `supervisord` 返回 `SIGCHLD` 信号的秒数。如果超过这个时间还没收到，则发送 `SIGKILL` 杀死进程。

##### stopasgroup

非必要参数，默认值为 `false` ，如果为 `true`，该标记会导致 `supervisor` 向整个进程组发送停止信号，并暗示 `killasgroup`选项 为 `true`。

##### killasgroup

非必要参数，默认值为 `false` ，如果为 `true`，则在向程序发送 `SIGKILL` 以终止程序时，会将其发送到整个进程组，同时也会照顾到其子进程，这在使用多进程程序中非常有用。

##### redirect_stderr

非必要参数，默认值为 `false` ，如果为 `true`，则会将进程的标准错误输出发送到标准输出文件描述符上，相当于用 `/the/program 2>&1`

##### stdout_logfile

非必要参数，默认值为 `AUTO` ，将进程的标准输出写入入此文件，如果未设置或设置为 `AUTO`，将自动选择文件位置并在重启时删除文件和备份。如果设置为 `NO`，将不创建日志文件。此值可以包含 Python 字符串表达式

##### stdout_logfile_maxbytes

非必要参数，默认值为 `50M` ，标准输出日志的滚动大小，可以带单位 KB, MB, GB ，如果为 `0` 则不滚动。

##### stdout_logfile_backups

非必要参数，默认值为 `10` ，标准输出日志的滚动保存数量，如果如果为 `0` 则不保存。

##### stdout_capture_maxbytes

非必要参数，默认值为 `0` ，当进程进入标准输出捕获模式(参见[Capture Mode](http://supervisord.org/logging.html#capture-mode))时，写入捕获 FIFO 的最大字节数。可以带单位 KB, MB, GB ，如果为 `0` 则关闭捕获模式。

##### stdout_events_enabled

非必要参数，默认值为 `false` ，如果为 `true`，则在进程向其标准输出文件描述符写入数据时，将触发 `PROCESS_LOG_STDOUT` 事件

>注：官方文档默认值写的是 `0` 但说明里面写的是 `if true`，这里以个人经验，对默认值做了处理。

##### stdout_syslog

非必要参数，默认值为 `false` ，如果为 `true`，写入进程标准输出的内容将连同进程名称一起发送到 syslog。

##### stderr_logfile

非必要参数，默认值为 `AUTO` ， 除非 `redirect_stderr` 为 `true`，否则将进程标准错误输出写入此文件。接受与 `stdout_logfile` 相同的值类型，并可包含相同的 Python 字符串表达式。

##### stderr_logfile_maxbytes

非必要参数，默认值为 `50MB` ，标准错误输出日志的滚动大小，可以带单位 KB, MB, GB ，如果为 `0` 则不滚动。

##### stderr_logfile_backups

非必要参数，默认值为 `10` ，标准错误输出日志的滚动时保存数量，如果如果为 `0` 则不保存。

##### stderr_capture_maxbytes

非必要参数，默认值为 `0` ，当进程进入标准错误输出捕获模式(参见[Capture Mode](http://supervisord.org/logging.html#capture-mode))时，写入捕获 FIFO 的最大字节数。可以带单位 KB, MB, GB ，如果为 `0` 则关闭捕获模式。

##### stderr_events_enabled

非必要参数，默认值为 `false` ，如果为 `true`，则在进程向其标准错误输出文件描述符写入数据时，将触发 `PROCESS_LOG_STDERR` 事件

##### stderr_syslog

非必要参数，默认值为 `false` ，如果为 `true`，写入进程标准错误输出的内容将连同进程名称一起发送到 syslog。

##### environment

非必要参数，没有默认值，可以设置为 `KEY="val",KEY2="val2"` 形式的键/值对列表。用于设置受控进程的环境变量。可以包含 Python 字符串表达式。

##### serverurl

非必要参数，默认值为 `AUTO` ，以 `SUPERVISOR_SERVER_URL` 环境变量传递给子进程，方便子进程与 supervisor 服务器通信。其语法和结构应与同名的 `[supervisorctl]` 部分选项相同。如果设置为 `AUTO`或未设置，supervisor 将自动构建一个服务器 URL，优先选择 UNIX 域套接字监听的服务器，而不是互联网套接字监听的服务器。

## 受控进程状态

STOPPED 已停止 指手工 stop 或者没有启动

STARTING 正在启动

RUNNING 正在运行

BACKOFF 进程进入STARTING状态，但很快就退出了，无法进入 RUNNING 状态

STOPPING 正在停止 

EXITED 已退出 指正常退出或者非正常退出

FATAL 进程无法成功启动

UNKNOWN 未知 一般指 supervisord 失败

### 状态图

参见：[Process States](http://supervisord.org/subprocess.html#process-states)