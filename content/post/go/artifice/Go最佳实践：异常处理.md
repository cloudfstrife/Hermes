---
title: "Go最佳实践：异常处理"
date: 2019-12-31T18:47:09+08:00
categories:
- Go
tags:
- Go
- error
keywords:
- Go
- error
---

记录一些go语言error处理的最佳实践

<!--more-->

#### 为package定义必要的错误类型

```go
type ErrSomething struct {
    message string
}

func (e *ErrZeroDivision) Error() string {
    return e.message
}
```

或者

```go
var ErrSomething = errors.New("Item not found")
```

这样做的好处是，当方法返回`error`时，调用者可以使用 `type switch` 或者 `errors.Is` 或者 `errors.As` 方法来判断错误类型，以分别进行处理。

#### 第三方package的函数返回的错误，立即wrap后再返回，自己编写的函数返回的错误，直接返回

这样做的好处是，当错误发生时，可以立即为错误添加错误发生时的上下文信息，并且可以防止错误重复wrap。

## 参考链接

[Go Best Practices — Error handling](https://medium.com/@sebdah/go-best-practices-error-handling-2d15e1f0c5ee)

[Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
