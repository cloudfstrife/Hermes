---
title: "Go语言bcrypt加密和校验"
date: 2019-12-18T19:03:07+08:00
categories:
- Go
tags:
- Go
- security
- encrypt
- bcrypt
keywords:
- Go
- security
- encrypt
- bcrypt
---

`bcrypt`是一个由Niels Provos以及David Mazières根据`blowfish`加密演算法所设计的密码杂凑函式，于1999年在USENIX中展示。实作中`bcrypt`会使用一个加盐的流程以防御彩虹表攻击，同时`bcrypt`还是适应性函式，它可以借由增加迭代次数来抵御日益增进的电脑运算能力的暴力法破解。[来自维基百科] 

<!--more-->

## golang.org/x/crypto/bcrypt

`golang.org/x/crypto/bcrypt`提供了Go语言的`bcrypt`实现，主要的函数有以下3个：

```go
// 密文校验
func CompareHashAndPassword(hashedPassword, password []byte) error
// 计算密文可能的cost值，主要用于检测哪些密码需要升级cost
func Cost(hashedPassword []byte) (int, error)
// 加密明文密码
func GenerateFromPassword(password []byte, cost int) ([]byte, error)
```

## 示例

```go
package main

import (
	log "github.com/sirupsen/logrus"
	"golang.org/x/crypto/bcrypt"
)

func init() {
	log.SetFormatter(&log.TextFormatter{
		FullTimestamp: true,
	})
	log.SetLevel(log.TraceLevel)
}

func main() {
	log.Info("Go")
	// 加密
	bls, err := bcrypt.GenerateFromPassword([]byte("Go"), 10)
	if err != nil {
		log.WithError(err).Error("bcrypt Error")
	}
	log.Infof("Bcrypt(\"Go\") = %s", string(bls))

	// 校验
	err = bcrypt.CompareHashAndPassword(bls, []byte("Go"))
	if err != nil {
		log.WithError(err).Error("bcrypt check Error")
	}

	// 计算Cost
	cost, err := bcrypt.Cost(bls)
	if err != nil {
		log.WithError(err).Error("bcrypt check Error")
	}
	log.Infof("Cost(\"%s\")=%d", string(bls), cost)
}
```

执行结果：


```text
time="2019-12-18T09:15:54+08:00" level=info msg=Go
time="2019-12-18T09:15:54+08:00" level=info msg="Bcrypt(\"Go\") = $2a$10$VCph3H9uCUt3huwd4q4Q9erga7RS88AHWJbsTBOGWgpFKXE0m1GOW"
time="2019-12-18T09:15:54+08:00" level=info msg="Cost(\"$2a$10$VCph3H9uCUt3huwd4q4Q9erga7RS88AHWJbsTBOGWgpFKXE0m1GOW\")=10"
```

## 参考链接

[bcrypt - 维基百科，自由的百科全书](https://zh.wikipedia.org/zh-hans/Bcrypt)
