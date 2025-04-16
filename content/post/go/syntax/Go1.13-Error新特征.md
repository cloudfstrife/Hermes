---
title: "Go1.13 Error新特征"
date: 2019-08-27T18:23:24+08:00
categories:
- Go
tags:
- Go
- error
keywords:
- Go
- error
---

即将发布的Go1.13对`errors`包进行了增强，新特征主要来自提案：[Proposal: Go 2 Error Inspection](https://go.googlesource.com/proposal/+/master/design/29934-error-values.md)。

<!--more-->

为了使`error`为程序员和程序提供足够多的相关信息，Go1.13对`error`处理的更新如下：

* 定义`Wrapper`接口（其实更应该称为`Unwrapper`）[\[1\]](#1)，用于Unwrap操作。
* `errors`包新增了`func Is(err, target error) bool`和`func As(err error, target interface{}) bool`函数，用于比较和断言error。
* 定义了`func Unwrap(err error) error`用于快速Unwrap错误。
* `fmt`包的`Errorf`方法修改了实现，增加新的verb`%w`，`%w`只接受error类型的参数，用于快速Wrap错误。

## 包装错误

```go
err := errors.New("my error")
err = fmt.Errorf("1s wrapping my error with Errorf: %w", err)
err = fmt.Errorf("2nd wrapping my error with Errorf: %w", err)
```

## 错误的解包

```go
err = errors.Unwrap(err)
```

> `func Unwrap(err error) error`间接调用err的`Unwrap() error`方法。

## 错误比较

```go
if errors.Is(err,os.PathError) {
	//Do Something
}
```

`Is`函数会逐层调用`Unwrap`函数并比较error链上的所有`error`是否与`target`匹配，直到匹配`targer`或者`Unwrap`返回`nil`。匹配`target`的条件：`Unwrap`返回值与`target`相同(==)或者`Unwrap`的返回值实现了`Is(error) bool`方法，调用`Is(target)`返回`true`。

## 错误断言

```go
if errors.As(err,&target){
	//Do Something
}
```

`As`函数会逐层调用`Unwrap`函数并比较error链上的所有`error`是否与`target`匹配，如果匹配，则把匹配到的值设置到`target`并回返`true`。  
如果`target`不是指向`error`类型或者`interface{}`类型的指针，则会`panic`，如果`err`为`nil`，返回`false`

## 示例

```go
package main

import (
	"errors"
	"fmt"
)

//BaseError is a Custom error
type BaseError struct {
	msg string
}

func (b BaseError) Error() string {
	return fmt.Sprintf("BaseError Message : %s", b.msg)
}

var baseError = BaseError{msg: "Base"}

func main() {
	err := fmt.Errorf("Wrap 001 : %w", baseError)
	err = fmt.Errorf("Wrap 002 : %w", err)
	fmt.Println(err)

	err = errors.Unwrap(err)
	fmt.Println(err)

	if errors.Is(err, baseError) {
		fmt.Println("err Is baseError")
	}

	var anotherError BaseError
	if errors.As(err, &anotherError) {
		fmt.Printf("%v\n", anotherError)
	}
}

```

执行结果：

```text
$ go run main.go
Wrap 002 : Wrap 001 : BaseError Message : Base
Wrap 001 : BaseError Message : Base
err Is baseError
BaseError Message : Base
```

## 建议

> 仅包装来自公共函数或方法的错误。否则就直接传播错误。

## 参考链接

[When to wrap errors](https://www.efekarakus.com/golang/2019/09/26/when-to-wrap-errors.html)

---

<span id="1">[1]</span> : 写这篇文章时，Go1.13版本还没有正式release，github仓库RC1分支的代码中没有`Wrapper`接口的定义，只是在`Unwrap`方法中引用了匿名的interface [参见](https://github.com/golang/go/blob/release-branch.go1.13/src/errors/wrap.go#L14)。  
之所以称之为`Wrapper`接口是因为[https://github.com/golang/xerrors/blob/master/wrap.go#L12](https://github.com/golang/xerrors/blob/master/wrap.go#L12)中定义的接口是`Wrapper`。


