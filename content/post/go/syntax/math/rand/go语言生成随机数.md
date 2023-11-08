---
title: "Go语言生成随机数"
date: 2019-05-05T12:44:00+08:00
categories:
- Go
tags:
- Go
- rand
keywords:
- Go
- rand
---

golang 内置能实现伪随机(math/rand)和真随机(crypto/rand)的库。

<!--more-->

##  真随机和伪随机

根据密码学原理，“随机数”随机性检验三个标准

* 统计学伪随机性：随机的比特流中，0和1的数量大致相等
* 密码学安全伪随机性：使用部分随机样本和随机数生成算法，**不可以**演算出随机样本的剩余部分。
* 真随机性：随机样本不可重现

只满足第一个标准的随机数称为**伪随机数**，同时满足前两个条件的随机数称为**密码学安全的伪随机数**，同时满足三个条件的随机数称为**真随机数**

## golang的伪随机

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	//初始化随机种子
	rand.Seed(time.Now().Unix())
	//生成100以内的伪随机数
	result := rand.Intn(100)
	fmt.Println(result)
}
```

## golang的真随机

```go
package main

import (
	"crypto/rand"
	"fmt"
	"math/big"
)

func main() {
    //生成100以内的伪随机数
	result, _ := rand.Int(rand.Reader, big.NewInt(100))
	fmt.Println(result)
}
```
