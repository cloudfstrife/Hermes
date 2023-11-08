---
title: "Visual Studio Code安装Go语言插件扩展工具"
date: 2018-12-10T14:51:09+08:00
categories:
- Go
tags:
- Go
- ide
- vscode
keywords:
- Go
- ide
- vscode
---

安装Visual Studio Code的Go语言插件所需要的扩展工具的shell

*基于现有的 Go module 代理 ，此脚本已经不再需要了。*
*基于现有的 Go module 代理 ，此脚本已经不再需要了。*
*基于现有的 Go module 代理 ，此脚本已经不再需要了。*

安装扩展工具，仅需要 `go env -w GOPROXY="https://goproxy.cn,direct"` ，然后就可以在 VS Code 中直接安装了。

<!--more-->

**Windows PowerShell** 

[install_vs_code_go_ext_tools.ps1](https://github.com/cloudfstrife/backup/blob/develop/shell/go/install_vs_code_go_ext_tools.ps1)

**Linux Shell**

[install_vs_code_go_ext_tools.sh](https://github.com/cloudfstrife/backup/blob/develop/shell/go/install_vs_code_go_ext_tools.sh)

> 注意:脚本依赖于同目录下的 [all_tools_information.csv](https://github.com/cloudfstrife/backup/blob/develop/shell/go/all_tools_information.csv) 文件
