---
title: "Ninja构建系统"
date: 2026-05-17T00:56:06+08:00
lastmod: 2026-05-17T00:56:06+08:00

categories:
  - other
tags:
  - ninja
keywords: 
 - ninja

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

Ninja构建系统

<!--more-->

## 简介

[Ninja构建系统][The Ninja build system] 是一个构建工具，用来调用各种工具（代码生成器、编译器、链接器等）来编译大型项目。

与常规的构建系统不同，Ninja在设计之初就**不是给人写的**，只是用于作为**其它程序生成的目标。**（即：由其它工具生成ninja文件，再使用ninja构建目标产物）。

Ninja的设计哲学就是避免任何模糊的内容，在 Makefile 里面可能经常会用`dir/*.cpp`来表示源文件，但是这需要搜索文件系统才能得到具体内容，是一个很慢而且很不确定的操作。Ninja把这些模糊的操作都交给元构建系统（meta-build system）来管理，本质上来说，Ninja就是分析目标产物的依赖关系，然后通过执行清晰的指令来构建目标产物。Ninja可以跟据依赖关系实现并行编译和增量编译。

## 安装

### Ninja

Ninja的[源码][ninja-build/ninja]可以从GitHub的仓库下载，同时提供了预编译的二进制文件，下载解压将可执行程序复制到`$PATH`目录即可。大部分Linux发行版的软件仓库也包含了Ninja，

```bash
# debian
sudo apt install ninja-build
```

### Meta-build system

#### [gn][GN]

gn 是 Google 的大型项目采用的编译系统，广泛应用在 chromium、Fuchsia 等项目中，主要的功能是生成 ninja 文件，再通过 ninja 编译真实的代码。

#### [CMake][CMake]

自CMake 2.8.8版本开始，可以在Linux系统上生成Ninja文件。新版本的CMake也支持在Windows和Mac OS X系统上生成Ninja文件。

#### [The Meson Build system][The Meson Build system]

Meson 是一个开源构建系统，旨在兼具极快的速度和尽可能友好的用户体验。

## Ninja文件

> 如上所说，Ninja在设计之初就**不是给人写的**，只是用于作为**其它程序生成的目标**。所以，在实际的开发工作中，一般使用元构建系统来生成Ninja文件即可。

> 以下内容仅用于熟悉Ninja文件的结构，除非想实现一个基于Ninja的元构建系统，否则，下面的内容并没什么用处。

Ninja运行时会分析ninja文件，生成依赖关系图，并根据文件修改时间来判定是否需要更新构建目标，执行必要的命令，这一点与Make工具相似。

ninja文件（默认名称：`build.ninja`）提供了一系列规则（rule）与构建语句（build）。规则是命令的简称，构建语句定义了如何使用规则来生成目标文件。以下是一个示例。

```ninja
# 指定版本依赖
ninja_required_version = 1.1
# 指定构建目录
builddir = output
# 定义变量
_version = v1.1.0

# 定义规则
rule show_version
    description = show version ${_version}
    # 使用`${参数名}` 多行命令以`$`连接
    # 使用`${in}`引用fwnmj的输入
    # 使用`${out}`引用规则的输出
    command = echo ${_version}; $
                echo ${in}; $
                echo ${out}

# 构建规则
build v1: phony

# 构建规则
build version : show_version v1                     
    _version = v1.2.0
# 上面的构建规则表示：使用 `show_version` 规则，传入 v1 做为规则的输入，构建出 version 目标产物，构建此产物时，将`_version`变量设置为`v1.2.0`

# 指定默认目标，多个default会被视为对默认目标列表的追加
default version
# 文件末尾的空行是必须的（MAC OS X平台，其它平台没有测试过）

```

### 变量

定义语法 `变量名 = 变量值` 

引用变量： 可以使用 `$变量名` 或者 `${变量名}` 来解引用

### 规则

规则语句用于声明命令的简称名称，定义格式如下：

```ninja
rule 规则名
    command = 命令
```

规则可以包含一系列的变量，除了 `command` 是必须的，其它变量都是可选的，完整的变量列表参见 [Rule variables][ref_rule]

#### phony 规则

特殊的规则 `phony`用于为其它目标创建一个别名。

```ninja
build foo: phony some/file/in/a/faraway/subdir/foo
```

当执行`ninja foo`时，将构建 `some/file/in/a/faraway/subdir/foo`目标。phony规则相当于一条不执行任何规则的规则，运行时不会被打印，不会被记录，也不记入构建命令计数。

### 构建

构建语句声明了输入文件与输出文件的关系，定义格式如下：

```ninja
build 输出文件列表: 规则名 输入文件列表
    变量名 = 变量值
```
构建语句跟随的带缩进的键值对用于在执行构建时，覆盖在文件中定义的其它变量。

## 运行构建

当在CLI中执行 `ninja` 时，ninja会在当前目录下寻找`build.ninja`文件，解析出依赖，如果文件中没有指定默认构建目标，则会构建所有不被其它build依赖的output。

```bash
# 构建 foo
ninja foo

# 使用 build-target.ninja 做为输入文件，构建 foo 
ninja -f build-target.ninja foo
```

[GN]: https://gn.googlesource.com/gn/+/main/README.md

[CMake]: https://cmake.org/

[The Meson Build system]: https://mesonbuild.com/

[ninja-build/ninja]:GitHub - ninja-build/ninja: a small build system with a focus on speed

[The Ninja build system]:https://ninja-build.org/

[ref_rule]: https://ninja-build.org/manual.html#ref_rule