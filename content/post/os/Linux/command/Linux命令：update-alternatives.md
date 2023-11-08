---
title: "Linux命令：update-alternatives"
date: 2022-07-25T17:11:01+08:00
categories:
- linux
- command
tags:
- linux
- command
- update-alternatives
keywords:
- linux
- command
- update-alternatives

---

`update-alternatives` 用于管理多个同功能的软件，或同一软件的不同版本间进行切换的工具，在不同的发行版中，命令参数略有差别，但使用方法大致相同。

<!--more-->

最经典的使用场景便是 `cc` ，在C语言项目中，使用 `cmake` 或者 `make` 编译时，编译命令往往不是直接使用 `gcc` 或者 `clang` ，而是使用 `cc` ， 在大部分 Linux 发行版中 `cc` 命令做为一个可执行程序，一般位于 `/usr/bin/cc` 

```text
$ ls -alh /usr/bin/cc
lrwxrwxrwx 1 root root 20 10月 10  2021 /usr/bin/cc -> /etc/alternatives/cc
```

这里的 `cc` 指向了 `/etc/alternatives/cc` ， 而 `/etc/alternatives/cc` 则指向了最终执行的命令 `/usr/bin/gcc` （当然，`/usr/bin/gcc` 也是个软链接，指具体的版本，这里不重点讨论）

```text
$ ls -alh /etc/alternatives/cc           
lrwxrwxrwx 1 root root 12 10月 10  2021 /etc/alternatives/cc -> /usr/bin/gcc
```

上面的示例，说明了 update-alternatives 的本质，即通过建立两重软链接的方式，在命令与实际的可执行程序间建立一个适配层。

## 用法

### 注册

```text
sudo update-alternatives --install link name path priority [ --slave slink sname spath]
```

选项说明

* link 一般为 `/usr/bin/`，`/usr/local/bin/` 等 `PATH` 环境变量查找命令的目录下的可执行程序，即上面示例的 `/usr/bin/cc`
* name `/etc/alternatives`目录中的链接名，这里只需要名称，不需要完整目录
* path 可执行程序的路径，需要完整的路径，并且必须是存在的文件
* priority 优先级，数字越大优先级越高。

示例

```text
sudo update-alternatives --install /usr/bin/testing testing /opt/testing/bin/testing 200
```

### 注册多个可执行程序

重复执行注册命令，即可将多个可执行程序注册到同一命令。

```text
sudo update-alternatives --install /usr/bin/testing testing $HOME/bin/testing 100
```

### 查看命令配置

```text
$ update-alternatives --display testing
testing - 自动模式
  最佳链接版本为 /home/xxxx/bin/testing
 链接目前指向 /home/xxxx/bin/testing
  链接 testing 指向 /usr/bin/testing
/home/xxxx/bin/testing - 优先级 300
/opt/testing/bin/testing - 优先级 200
```

这里的第一行，表示 testing 命令由 update-alternatives 以自动模式（默认）管理，自动模式使用优先级高的可执行程序。

### 交互式配置

```text
$ sudo update-alternatives --config testing
有 2 个候选项可用于替换 testing (提供 /usr/bin/testing)。

  选择       路径                     优先级  状态
------------------------------------------------------------
* 0            /home/xxxx/bin/testing   300       自动模式
  1            /home/xxxx/bin/testing   300       手动模式
  2            /opt/testing/bin/testing    200       手动模式

要维持当前值[*]请按<回车键>，或者键入选择的编号：2
update-alternatives: 使用 /opt/testing/bin/testing 来在手动模式中提供 /usr/bin/testing (testing)
```

上面的命令使用交互式配置 testing 可执行程序，选择第二个可执行程序做为 testing 命令的可执行程序。再次执行 `update-alternatives --display testing` 可以看到 `testing` 已经修改为手动模式。

### 非交互式配置

```text
$ sudo update-alternatives --set testing /opt/testing/bin/testing
```

### 删除

```text
$ sudo update-alternatives --remove testing $HOME/bin/testing
```

当命令注册的所有可执行程序被删除时，命令也将被删除。

> **小技巧**
> 
> `update-alternatives` 的本质是利用 Linux 系统的软链实现版本管理，所以 `update-alternatives` 是可以处理目录的。

比如在管理多个版本的 JDK 时 可以创建一个指向目录的项，这里以 jdk_home 为例

```text
$ sudo update-alternatives --install /usr/local/jdk jdk_home /opt/testing/jdk1.7 100
$ sudo update-alternatives --install /usr/local/jdk jdk_home /opt/testing/jdk1.8 200
$ sudo update-alternatives --install /usr/local/jdk jdk_home /opt/testing/jdk1.9 300
$ ls -alh /usr/local/jdk       
lrwxrwxrwx 1 root root 26  7月 25 15:23 /usr/local/jdk -> /etc/alternatives/jdk_home
$ ls -alh /etc/alternatives/jdk_home              
lrwxrwxrwx 1 root root 19  7月 25 15:23 /etc/alternatives/jdk_home -> /opt/testing/jdk1.9
```

此时配置 `JAVA_HOME` 为 `/usr/local/jdk` 即可使用 `update-alternatives` 轻松管理 JDK 版本，即使切换版本，也不需要注销后重新登陆。