---
title: "Go crypto/cipher"
date: 2019-07-29T18:47:49+08:00
categories:
- golang
- package
tags:
- golang
- package
- crypto
- cipher
keywords:
- golang
- package
- crypto
- cipher
---

`crypto/cipher`实现了标准块密码模式，可以围绕低级块密码实现。  
详见[ https://csrc.nist.gov/groups/ST/toolkit/BCM/current_modes.html ](https://csrc.nist.gov/groups/ST/toolkit/BCM/current_modes.html) 和 NIST Special Publication 800-38A

<!--more-->

## 分组加密

分组加密（块加密）: [分组密码](https://zh.wikipedia.org/wiki/%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81)

分组密码工作模式: [分组密码工作模式](https://zh.wikipedia.org/wiki/%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F#%E5%88%9D%E5%A7%8B%E5%8C%96%E5%90%91%E9%87%8F%EF%BC%88IV%EF%BC%89)



## 参考资料


[Block cipher mode of operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation)  
[Encrypting Streams in Go](https://medium.com/blend-engineering/encrypting-streams-in-go-6cff6062a107)  
[Package cipher ](https://golang.org/pkg/crypto/cipher/)  
[stream.go](https://github.com/blend/go-sdk/blob/master/crypto/stream.go)  