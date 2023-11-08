---
title: "Go语言自定义struct字段标签"
date: 2019-12-24T18:43:34+08:00
categories:
- Go
tags:
- Go
keywords:
- Go
---

在Go语言当中，可以为struct添加自定义标签，以优雅简单的方式存储字段的元数据(例如：ORM映射，数据校验类型等等)。标签信息都是静态的，不需要实例化struct也可以通过反射获取到。 

<!--more-->

## 语法

```go
type struct_name struct {
	field_name field_type `tag_key_1:"tag_value_1" tag_key_2:"tag_value_2"`
}
```

> struct标签由一个或多个键值对组成，键与值使用冒号分隔，值用双引号包裹，多个键值对之间使用空格分隔。 


## 示例

```go
package main

import (
	"reflect"

	log "github.com/sirupsen/logrus"
)

func init() {
	log.SetFormatter(&log.TextFormatter{
		FullTimestamp: true,
	})
}

//Testing test struct
type Testing struct {
	Email string `validate:"email;not_null" json:"email"`
}

func main() {
	var testing Testing
	t := reflect.TypeOf(testing)

	f, b := t.FieldByName("Email")
	if b {
		jsonField := f.Tag.Get("json")
		log.Info(jsonField)
		validateField := f.Tag.Get("validate")
		log.Info(validateField)
	}
}

```

编译与运行

```text
$ go build
$ ./testing.exe
time="2019-12-25T17:38:30+08:00" level=info msg=email
time="2019-12-25T17:38:30+08:00" level=info msg="email;not_null"

```
