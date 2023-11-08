---
title: "Go语言单元测试常规操作"
date: 2019-05-10T18:57:56+08:00
categories:
- Go
tags:
- Go
- testing
keywords:
- Go
- testing
---

本文以一个返回斐波纳切数列的函数做为示例，记录Go程序单元测试，性能测试的编写。

<!--more-->

## 环境说明

| 项目     | 说明                              |
| -------- | --------------------------------- |
| 操作系统 | Microsoft Windows 10 家庭中文版   |
| go版本   | go version go1.12.5 windows/amd64 |
| CLI      | git version 2.19.2.windows.1      |

## Go Testing

### 编写函数

```text
$ go mod init app/testing
$ mkdir -p fibonacci
$ touch fibonacci/fibonacci.go
```

**fibonacci/fibonacci.go**

```go
package fibonacci

//Fibonacci 返回n以内的斐波那契数列元素
func Fibonacci(n int64) []int64 {
	if n < 0 {
		return []int64{}
	}
	result := []int64{1, 1}
	if n <= 1 {
		return result
	}
	c := result[0] + result[1]
	for i := int64(2); c <= n; i, c = i+1, result[i]+result[i-1] {
		result = append(result, c)
	}
	return result
}
```

### 编写测试代码

```text
$ touch fibonacci/fibonacci_test.go
```

**fibonacci/fibonacci_test.go**

```go
package fibonacci

import (
	"fmt"
	"reflect"
	"testing"
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
			want:  []int64{1, 1},
		},
		"value 5  million": {
			input: 5000000,
			want:  []int64{1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181, 6765, 10946, 17711, 28657, 46368, 75025, 121393, 196418, 317811, 514229, 832040, 1346269, 2178309, 3524578},
		},
	}

	for CaseName, CaseVal := range TestCases {
		t.Run(CaseName, func(t *testing.T) {
			got := Fibonacci(CaseVal.input)
			if !reflect.DeepEqual(got, CaseVal.want) {
				t.Errorf("test case [ %s ] faild : want :%#v got :%#v", CaseName, CaseVal.want, got)
			}
		})
	}
}
func BenchmarkFibonacci(b *testing.B) {
	var i int64 = 1000000
	for index := 0; index < b.N; index++ {
		Fibonacci(i)
	}
}

func ExampleFibonacci() {
	fmt.Println(Fibonacci(1))
	//Output:[1 1]
}
```

go语言的测试代码应该与被测试代码放在相同的包里，测试源代码文件以`_test.go`结尾。

**单元测试**函数签名格式为`func TestXxx(*testing.T)`，函数名以`Test`开头，输入参数为`testing.T`类型的指针。

**基准测试**函数签名格式为`func BenchmarkXxx(*testing.B)`，函数以`Benchmark`开头，输入参数为`testing.B`类型的指针。

基准函数必须运行b.N次被测试代码。在执行基准测试期间，go test命令会先尝试把b.N设置为1，并执行测试函数，如果测试执行时间没有超过执行时间上限（默认1秒），则会改大b.N的值，并再次测试。直到这个时间大于等于上限为止。

**注意** b.N是被测试函数执行的次数，不是Benchmark函数的执行次数。

**示例代码**函数签名格式为`func ExampleXxx()`，函数体结尾处可以包含以`// Output:`开头的输出验证信息。如果不包含此注释，运行测试时，此示例不会运行。

在上面的代码中，使用了一些技巧。

> * `TestFibonacci`函数中，创建了一个map，map的key为测试用例名称，value为一个匿名struct，此struct属性包含一个int64的输入参数和一个int64切片的预期结果。执行测试时，循环map的元素，通过`t.Run()`启动子测试，调用被测试函数，并比较测试结果是否与预期结果相同。这样写的好处是不需要为每个测试用例编写一个方法，同时减少代码冗余。当需要新增测试用例时，只需要向map中添加新的元素，重新测试即可。  
> * 在结果验证时，当验证失败，使用`t.Errorf`而不使用`t.Fatalf`，这样的写法防止一个测试用例失败，后面的测试用例就不执行了。执行一次测试，即可看出哪些用例失败。  
> * 在输出时，使用`%#v`而不是`%v`，这样的输出，可以更清晰，更可读的输出结果差异。  

## 执行单元测试

```text
$ go test -v app/testing/fibonacci
```

输出：

```text
$ go test -v app/testing/fibonacci
=== RUN   TestFibonacci
--- PASS: TestFibonacci (3.00s)
=== RUN   ExampleFibonacci
--- PASS: ExampleFibonacci (1.00s)
PASS
ok      app/testing/fibonacci   4.051s
```

修改`TestFibonacci`中的`map`的预期数据，并去除`ExampleFibonacci`的`//Output:[1]`再次执行

```text
$ go test -v app/testing/fibonacci
=== RUN   TestFibonacci
--- FAIL: TestFibonacci (3.00s)
    fibonacci_test.go:31: test case [ negative ] faild : want :[]int64{1} got :[]int64{}
    fibonacci_test.go:31: test case [ value one ] faild : want :[]int64{1, 2} got :[]int64{1}
FAIL
FAIL    app/testing/fibonacci   3.057s
```

## 基准测试

```text
$ go test -bench . -run ^$ ./fibonacci
```

> * `-bench`指定执行哪些基准测试，`.`表示测试此包中下的所有基准测试，可以使用正则表达式来执行特定的测试
> 
> * `-run`指定执行哪些单元测试，此处的`^$`表示执行名称为空的测试，即不执行任何单元测试。
> 
> * windows 的cmd命令行下要实现以上效果，命令有点不一样
> 
> `go test -v -bench="." -run="^$" .\fibonacci` 
> 

输出

```text
$ go test -bench . -run ^$ ./fibonacci
goos: windows
goarch: amd64
pkg: app/testing/fibonacci
BenchmarkFibonacci-8     5000000               396 ns/op
PASS
ok      app/testing/fibonacci   2.302s
```

## 参考链接

[testing - The Go Programming Language](https://golang.org/pkg/testing/)
