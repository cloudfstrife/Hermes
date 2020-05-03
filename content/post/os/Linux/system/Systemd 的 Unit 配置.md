---
title: "systemd的 Unit 配置"
date: 2020-01-15T18:53:05+08:00
categories:
- Linux
- system
tags:
- linux
- init
- systemd
keywords:
- linux
- init
- systemd
---

systemd 包含一系列工具集合，其作用不仅包含初始化操作系统，还包括后台服务管理，日志归档，设备管理，电源管理、定时任务等

<!--more-->

systemd 将不同的资源统称为 Unit (单元) ，Unit 配置文件按照约定，放在指定的系统目录下

* `/etc/systemd/system`			&emsp;&emsp;&emsp;&emsp;&emsp;系统或用户自定义的配置文件
* `/run/systemd/system`			&emsp;&emsp;&emsp;&emsp;&emsp;软件运行时生成的配置文件
* `/usr/lib/systemd/system`		&emsp;&emsp;&emsp;&emsp;系统或第三方软件安装时添加的配置文件。

目录有优先级之分，按照上面的顺序优先级依次递减，当三个目录中有同名文件时，优先级较高的目录里的文件会被优先使用使用

## Unit文件格式

systemd 服务的 Unit 文件可以分三个段

* Unit
* Service 
* Install

以下是ssh.service

```text
$ cat /usr/lib/systemd/system/ssh.service 
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
Alias=sshd.service

```

### Unit

Unit 段用于定义 Unit 以及 Unit 的依赖关系，主要的配置属性有以下几个：

| 属性名        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| Description   | 描述这个 Unit 文件的信息                                     |
| Documentation | 指定服务的文档，可以是一个或多个文档的 URL 路径              |
| Requires      | 依赖的其它 Unit 列表，列在其中的 Unit 会在这个Unit启动时的同时被启动，如果其中任意一个Unit启动失败，这个Unit也会启动失败。 |
| Wants         | 与 Requires 相似，但只是在当前 Unit 启动时，触发启动列出的 Unit ，而不考虑这些 Unit 启动是否成功 |
| After         | 与 Requires 相似，但是在后面列出的所有 Unit 全部启动后，才会启动当前的 Unit |
| Before        | 与 After 相反，当前 Unit 应该在列出的 Unit 之前启动          |
| OnFailure     | Unit 启动失败时，自动启动列出的每个 Unit                     |
| Conflicts     | 与这个 Unit 有冲突的模块，如果列出的 Unit 已经在运行时，当前 Unit 不能启动，反之亦然 |

### Service 

Service 段用于定义服务型（以`.service`结尾） Unit 的启动行为，主要的配置属性有以下几个：

| 属性名           | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| EnvironmentFile  | 指定当前服务的环境变量定义文件，该文件内部的 `key=value` 键值对，可以用 `$key` 的形式，在当前`.service`文件中引用 |
| ExecStart        | 定义启动进程时执行的命令                                     |
| ExecStartPre     | 启动当前服务之前执行的命令                                   |
| ExecStartPost    | 启动当前服务之后执行的命令                                   |
| ExecStop         | 停止服务时执行的命令                                         |
| ExecStopPost     | 停止服务之后执行的命令                                       |
| ExecReload       | 重启服务时执行的命令                                         |
| Type             | 启动类型，取值见下表 [启动类型](#启动类型)                   |
| WorkingDirectory | 指定服务的工作目录                                           |
| RootDirectory    | 指定服务进程的根目录，如果配置了这个参数，服务将无法访问指定目录以外的文件 |
| User             | 指定运行服务的用户                                           |
| Group            | 指定运行服务的用户组                                         |
| KillMode         | 定义 Systemd 如何停止服务，取值见下表 [KillMode](#killmode)  |

> 所有的启动属性之前，都可以加上一个减号（-），表示 "抑制错误" ，即发生错误的时候，不影响其他命令的执行。

#### 启动类型

| 类型    | 说明                                                   |
| ------- | ------------------------------------------------------ |
| simple  | 默认值，执行ExecStart指定的命令，启动主进程            |
| forking | 以 fork 方式从父进程创建子进程，创建后父进程会立即退出 |
| oneshot | 一次性进程，Systemd 会等当前服务退出，再继续往下执行   |
| dbus    | 当前服务通过D-Bus启动                                  |
| notify  | 当前服务启动完毕，会通知Systemd，再继续往下执行        |
| idle    | 若有其他任务执行完毕，当前服务才会运行                 |

#### KillMode

| 类型                    | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| control-group           | 默认值,当前控制组里面的所有子进程，都会被杀掉      |
| process                 | 只杀主进程                                         |
| mixed                   | 主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号 |
| none                    | 没有进程会被杀掉，只是执行服务的 stop 命令。       |

### Install

Install 段用于定义如何安装这个配置文件，使 Unit 在系统启动时自动运行，主要的配置属性有以下几个：

| 属性名     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| WantedBy   | 值为一个或多个 Target，当前 Unit 激活 (enable) 时符号链接会放入 `/usr/lib/systemd/system` 目录下以 `<Target>.wants` 后缀构成的子目录中 |
| Also       | 当前 Unit enable/disable 时，同时 enable/disable 的其他 Unit |
| Alias      | 当前 Unit 可用于启动的别名                                   |



