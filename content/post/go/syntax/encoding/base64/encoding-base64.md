---
title: "Go encoding/base64"
date: 2020-04-12T20:57:25+08:00
categories:
- Go
tags:
- Go
- encoding
- base64
keywords:
- Go
- encoding
- base64
---

Go 语言内置的 `encoding/base64` 提供了 base64 编码与解码功能。主要方法见 [base64 package · go.dev](https://pkg.go.dev/encoding/base64?tab=doc)

<!--more-->

## 内置的 Encoding

* **StdEncoding** —— 常规编码器
* **URLEncoding** —— URL安全编码器
* **RawStdEncoding** —— 常规编码器，末尾不补=
* **RawURLEncoding** —— URL编码器，末尾不补=

## base64 编码

```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	original := "Go"
	encoded := base64.StdEncoding.EncodeToString([]byte(original))
	fmt.Println(encoded)
}
```

## base64 解码

```go
package main

import (
	"encoding/base64"
	"fmt"
	"log"
)

func main() {
	decoded, err := base64.StdEncoding.DecodeString("R28=")
	if err != nil {
		log.Println(err)
	}
	fmt.Println(string(decoded))
}
```

## URL base64编解码

```go
package main

import (
	"encoding/base64"
	"fmt"
	"log"
)

func main() {
	uEnc := base64.URLEncoding.EncodeToString([]byte("Hello World"))
	fmt.Println(uEnc)

	uDec, err := base64.URLEncoding.DecodeString(uEnc)
	if err != nil {
		log.Println(err)
	}
	fmt.Println(string(uDec))
}
```
