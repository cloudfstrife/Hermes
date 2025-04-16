---
title: "Go Module使用私有仓库遇到的问题及解决方案"
date: 2020-10-08T11:40:02+08:00
categories:
- Go
tags:
- Go 
keywords:
- go
---

本文记录一些 Go Module 私有库引用中遇到的问题及解决方案

<!--more-->

## Go Module 私有库配置

**Go Module使用私有库的基础操作**

```text
$ go env -w GOPRIVATE="*.corp.example.com,rsc.io/private"
```

`GOPRIVATE` 用来控制 `go` 命令把哪些仓库看做是私有的仓库，在处理匹配的库时，会跳过 `GOPROXY` 代理和校验检查。

## 私有库 authenticate

一般私有仓库都需要鉴权，`go` 命令在拉取或者构建时，会报如下的错误

```text
fatal: could not read Username for 'https://xxxx.xxx': terminal prompts disabled
```

解决办法有两种

### 方法一

```text
$ export GIT_TERMINAL_PROMPT=1
$ go build 
```

期间会提示输入用户名和密码

### 方法二

* 在 git 的个人账户生成一个 Person Access Token

	> [github - Creating a personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)  
	> [gitlab - Personal access tokens](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)

* 创建或者附加 Person Access Token 配置到 `~/.netrc`

```text
cat << EOF| tee -a ~/.netrc
machine github.com login USERNAME password APIKEY
EOF
```

> 替换上面的域名 ， USERNAME ， APIKey 为生成的 Person Access Token

## 私有仓库不支持 https

一般会有错误提示：

```text
x509: certificate signed by unknown authority
```

解决方法：

```text
go get -insecure xxx
```

`-insecure`会以不安全的方式( http )获取仓库内容。

如果构建时报这个错，解决方案是设置 `go env`

```text
go env -w GOINSECURE="*.corp.example.com,rsc.io/private"
```

这种方法是针对匹配的库以不安全的方式( http )获取，其它库以安全方式获取。

以上就是个人在工作过程中遇到的关于使用私有库的问题，算是一个总结，希望可以帮助到正在阅读的你。

祝好 ^_~