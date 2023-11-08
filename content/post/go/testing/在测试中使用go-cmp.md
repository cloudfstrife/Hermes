---
title: "在测试中使用go-cmp"
date: 2019-05-13T19:40:21+08:00
categories:
- Go
tags:
- Go
- testing
- go-cmp
keywords:
- Go 
- testing
- go-cmp
---

`go-cmp`用于比较两个值是否相等,相比`reflect.DeepEqual`，它更安全，更强大。

`go-cmp`基于BSD License发布，可以放心使用。

<!--more-->

> 本文中的部分示例，使用 [go语言单元测试常规操作](/2019/05/go语言单元测试常规操作) 中的示例。

## Equal

函数签名：

```go
func Equal(x, y interface{}, opts ...Option) bool
```

Equal函数用于比较两个值是否相等。其会经过以下规则：

1. 在经过路径过滤，值过滤和类型过滤之后，会生一些忽略、转换、比较选项，如果选项中存在忽略，则忽略比较，如果转换器和比较器的数据大于1，则会panic（因为比较操作不明确）。如果选项中存在转换器，则调用转换器转换当前值，再递归调用转换器输出类型的Equal。如果包含一个比较器。则比较使用比较器比较当前值。否则进入下一比较阶段。
1. 如果比较值有一个`(T) Equal(T) bool` 或者 `(T) Equal(I) bool`，那么，即使x与y是nil，也会调用`x.Equal(y)`做为结果。如果不存在这样的方法，则进入下一阶段。
1. 在最后阶段，`Equal`方法尝试比较x与y的基本类型。使用go语言的 `==` 比较基本类型（bool, intX, float32,float64, complex32,complex64, string, chan）。

在比较struct时，将递归的比较struct的字段。如果结构体包含未导出的字段，函数会panic。可以通过指定`cmpopts.IgnoreUnexported`来忽略未导出的字段，也可以使用`cmp.AllowUnexported`来指定比较未导出的字段。

### 示例

#### 存在未导出字段

```go
package fibonacci

import (
	"testing"

	"github.com/google/go-cmp/cmp"
)

func TestEqual(t *testing.T) {
	type User struct {
		UserName string
		password string
	}

	u1 := User{
		UserName: "U1",
		password: "password_for_u1",
	}

	u2 := User{
		UserName: "U1",
		password: "password_for_u2",
	}

	if !cmp.Equal(u1, u2) {
		t.Errorf("u1 is not equals u2")
	}
}

```

运行结果 

```text
$ go test -v -run TestEqual app/testing/fibonacci
=== RUN   TestEqual
--- FAIL: TestEqual (0.00s)
panic: cannot handle unexported field: {fibonacci.User}.password
consider using a custom Comparer; if you control the implementation of type, you can also consider AllowUnexported or cmpopts.IgnoreUnexported [recovered]
        panic: cannot handle unexported field: {fibonacci.User}.password
consider using a custom Comparer; if you control the implementation of type, you can also consider AllowUnexported or cmpopts.IgnoreUnexported

goroutine 19 [running]:
testing.tRunner.func1(0xc000104100)
        D:/go/src/testing/testing.go:830 +0x399
panic(0x55df20, 0xc000056570)
        D:/go/src/runtime/panic.go:522 +0x1c3
github.com/google/go-cmp/cmp.validator.apply(0xc0000f2080, 0x55df20, 0xc00005c590, 0xb8, 0x55df20, 0xc00005c5b0, 0xb8)
        E:/go/env/pkg/mod/github.com/google/go-cmp@v0.3.0/cmp/options.go:229 +0x162
github.com/google/go-cmp/cmp.(*state).tryOptions(0xc0000f2080, 0x5ce4a0, 0x55df20, 0x55df20, 0xc00005c590, 0xb8, 0x55df20, 0xc00005c5b0, 0xb8, 0x8)
        E:/go/env/pkg/mod/github.com/google/go-cmp@v0.3.0/cmp/compare.go:269 +0x130
github.com/google/go-cmp/cmp.(*state).compareAny(0xc0000f2080, 0x5cc4c0, 0xc000104200)
        E:/go/env/pkg/mod/github.com/google/go-cmp@v0.3.0/cmp/compare.go:224 +0x285
github.com/google/go-cmp/cmp.(*state).compareStruct(0xc0000f2080, 0x5ce4a0, 0x576e00, 0x576e00, 0xc00005c580, 0x99, 0x576e00, 0xc00005c5a0, 0x99)
        E:/go/env/pkg/mod/github.com/google/go-cmp@v0.3.0/cmp/compare.go:383 +0x588
github.com/google/go-cmp/cmp.(*state).compareAny(0xc0000f2080, 0x5cc300, 0xc00005e840)
        E:/go/env/pkg/mod/github.com/google/go-cmp@v0.3.0/cmp/compare.go:252 +0x1014
github.com/google/go-cmp/cmp.Equal(0x576e00, 0xc00005c580, 0x576e00, 0xc00005c5a0, 0x0, 0x0, 0x0, 0x1)
        E:/go/env/pkg/mod/github.com/google/go-cmp@v0.3.0/cmp/compare.go:107 +0x3be
app/testing/fibonacci.TestEqual(0xc000104100)
        E:/space/testing/fibonacci/fibonacci_test.go:69 +0xfa
testing.tRunner(0xc000104100, 0x5a6550)
        D:/go/src/testing/testing.go:865 +0xc7
created by testing.(*T).Run
        D:/go/src/testing/testing.go:916 +0x361
FAIL    app/testing/fibonacci   0.063s
```

#### 指定比较未导出字段

```go
package fibonacci

import (
	"testing"

	"github.com/google/go-cmp/cmp"
)

func TestEqual(t *testing.T) {
	type User struct {
		UserName string
		password string
	}

	u1 := User{
		UserName: "U1",
		password: "password_for_u1",
	}

	u2 := User{
		UserName: "U1",
		password: "password_for_u2",
	}

	opts := []cmp.Option{
		cmp.AllowUnexported(u1, u2),
	}

	if !cmp.Equal(u1, u2, opts...) {
		t.Errorf("u1 is not equals u2")
	}
}

```

执行结果

```text
$ go test -v -run TestEqual app/testing/fibonacci
=== RUN   TestEqual
--- FAIL: TestEqual (0.00s)
    fibonacci_test.go:70: u1 is not equals u2
FAIL
FAIL    app/testing/fibonacci   0.057s
```

#### 忽略未导出字段 

```go
package fibonacci

import (
	"testing"

	"github.com/google/go-cmp/cmp"
	"github.com/google/go-cmp/cmp/cmpopts"
)

func TestEqual(t *testing.T) {
	type User struct {
		UserName string
		password string
	}

	u1 := User{
		UserName: "U1",
		password: "password_for_u1",
	}

	u2 := User{
		UserName: "U1",
		password: "password_for_u2",
	}

	opts := []cmp.Option{
		cmpopts.IgnoreUnexported(u1, u2),
	}

	if !cmp.Equal(u1, u2, opts...) {
		t.Errorf("u1 is not equals u2")
	}
}
```

执行结果 

```text
$ go test -v -run TestEqual app/testing/fibonacci
=== RUN   TestEqual
--- PASS: TestEqual (0.00s)
PASS
ok      app/testing/fibonacci   0.068s

```

#### 自定义比较器

```go
package fibonacci

import (
	"testing"

	"github.com/google/go-cmp/cmp"
)

func TestEqual(t *testing.T) {
	type User struct {
		UserName string
		password string
	}

	u1 := User{
		UserName: "U1",
		password: "password_for_user",
	}

	u2 := User{
		UserName: "U2",
		password: "password_for_user",
	}

	opts := []cmp.Option{
		cmp.Comparer(func(x, y User) bool {
			return u1.password == u2.password
		}),
	}

	if !cmp.Equal(u1, u2, opts...) {
		t.Errorf("u1 is not equals u2")
	}
}

```

运行结果 

```text
$ go test -v -run TestEqual app/testing/fibonacci
=== RUN   TestEqual
--- PASS: TestEqual (0.00s)
PASS
ok      app/testing/fibonacci   (cached)

```

#### 自定义转换器


```go
package fibonacci

import (
	"testing"

	"github.com/google/go-cmp/cmp"
)

func TestEqual(t *testing.T) {
	type User struct {
		UserName string
		password string
	}

	u1 := User{
		UserName: "U1",
		password: "password_for_user",
	}

	u2 := User{
		UserName: "U2",
		password: "password_for_user",
	}

	opts := []cmp.Option{
		cmp.Transformer("Transfor_user", func(x User) string {
			return x.UserName
		}),
	}

	if !cmp.Equal(u1, u2, opts...) {
		t.Errorf("u1 is not equals u2")
	}
}

```

运行结果

```text
$ go test -v -run TestEqual app/testing/fibonacci
=== RUN   TestEqual
--- FAIL: TestEqual (0.00s)
    fibonacci_test.go:72: u1 is not equals u2
FAIL
FAIL    app/testing/fibonacci   0.062s
```

#### struct的Equal函数 

```go
package fibonacci

import (
	"testing"

	"github.com/google/go-cmp/cmp"
)

type User struct {
	UserName string
	password string
}

func (u User) Equal(c User) bool {
	return u.password == c.password
}

func TestEqual(t *testing.T) {
	u1 := User{
		UserName: "U1",
		password: "password_for_user",
	}

	u2 := User{
		UserName: "U2",
		password: "password_for_user",
	}

	if !cmp.Equal(u1, u2) {
		t.Errorf("u1 is not equals u2")
	}
}
```

运行结果 


```text
$ go test -v -run TestEqual app/testing/fibonacci
=== RUN   TestEqual
--- PASS: TestEqual (0.00s)
PASS
ok      app/testing/fibonacci   (cached)
```


#### option与Equal方法同时存在的情况

```go
package fibonacci

import (
	"testing"

	"github.com/google/go-cmp/cmp"
)


type User struct {
	UserName string
	password string
}

func (u User) Equal(c User) bool {
	return u.password == c.password
}

func TestEqual(t *testing.T) {
	u1 := User{
		UserName: "U1",
		password: "password_for_user",
	}

	u2 := User{
		UserName: "U2",
		password: "password_for_user",
	}

	opts := []cmp.Option{
		cmp.Transformer("Transfor_user", func(x User) string {
			return x.UserName
		}),
	}

	if !cmp.Equal(u1, u2, opts...) {
		t.Errorf("u1 is not equals u2")
	}
}
```

运行结果

```text
$ go test -v -run TestEqual app/testing/fibonacci
=== RUN   TestEqual
--- FAIL: TestEqual (0.00s)
    fibonacci_test.go:76: u1 is not equals u2
FAIL
FAIL    app/testing/fibonacci   0.089s

```

> 解释：比较操作第一阶段，因为存在匹配的Transformer选项，所以使用Transformer转换`u1`和`u2`并比较转换后的字符串，即`u1`的`UserName`和`u2`的`UserName`，所以比较结果为`false`


## Diff

函数签名：

```go
func Diff(x, y interface{}, opts ...Option) string
```

Diff 函数返回人类可读的两个值的比较结果。当且仅当输入的值相同时，才返回空字符串。

在输出结果中，以`-`开头的行表示x中有y中没有的元素，以`+`开头的行表示y中有x中没有元素。

**示例**

```go
package fibonacci

import (
	"fmt"
	"testing"

	"github.com/google/go-cmp/cmp"
)

func TestFibonacci(t *testing.T) {
	TestCases := map[string]struct {
		input int64
		want  []int64
	}{
		"negative": {
			input: -1,
			want:  []int64{},
		},
		"value one": {
			input: 1,
			want:  []int64{1},
		},
		"value 5  million": {
			input: 5000000,
			want:  []int64{1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 1597, 2584, 4181, 6765, 10946, 17711, 28657, 46368, 75025, 121393, 196418, 317811, 514229, 832040, 1346269, 2178309, 3524578, 5702887},
		},
	}

	for _, CaseVal := range TestCases {
		got := Fibonacci(CaseVal.input)
		if diff := cmp.Diff(CaseVal.want, got); diff != "" {
			t.Error(diff)
		}
	}
}
```

> 修改内容：测试用例`value 5  million`去除一个元素，for循环部分代码修改。

```text
$ go test -v  app/testing/fibonacci
=== RUN   TestFibonacci
--- FAIL: TestFibonacci (0.00s)
    fibonacci_test.go:32:   []int64{
                ... // 12 identical elements
                377,
                610,
        +       987,
                1597,
                2584,
                ... // 16 identical elements
          }
=== RUN   ExampleFibonacci
--- PASS: ExampleFibonacci (0.00s)
FAIL
FAIL    app/testing/fibonacci   0.053s
```

> 输出的含义代表`got`的结果比预期结果多了一个值为`987`的元素。

## 参考链接

[cmp - GoDoc](https://godoc.org/github.com/google/go-cmp/cmp)

[GitHub - google/go-cmp: Package for comparing Go values in tests](https://github.com/google/go-cmp)
