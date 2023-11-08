---
title: "Go语言单元测试非常规操作"
date: 2019-05-11T19:01:30+08:00
categories:
- Go
tags:
- Go
- testing
keywords:
- Go
- testing
---

Go语言单元测试非常规操作，本文的部分示例命令使用 [go语言单元测试常规操作](/2019/05/go语言单元测试常规操作) 中的示例。

<!--more-->

## 子测试和子基准

```go
func TestFibonacci(t *testing.T) {
	t.Run("negative", func(t *testing.T) {
		got := Fibonacci(-1)
		if !reflect.DeepEqual(got, []int64{}) {
			t.Errorf("test case [ %s ] faild : want :%#v got :%#v", "negative", []int64{}, got)
		}
	})
	t.Run("value one", func(t *testing.T) {
		got := Fibonacci(1)
		if !reflect.DeepEqual(got, []int64{1, 1}) {
			t.Errorf("test case [ %s ] faild : want :%#v got :%#v", "value one", []int64{0, 1, 1}, got)
		}
	})
	t.Run("value 5 million", func(t *testing.T) {
		got := Fibonacci(5000000)
		if !reflect.DeepEqual(got, []int64{1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181, 6765, 10946, 17711, 28657, 46368, 75025, 121393, 196418, 317811, 514229, 832040, 1346269, 2178309, 3524578}) {
			t.Errorf("test case [ %s ] faild : want :%#v got :%#v", "value 5  million", []int64{0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181, 6765, 10946, 17711, 28657, 46368, 75025, 121393, 196418, 317811, 514229, 832040, 1346269, 2178309, 3524578}, got)
		}
	})
}
```

`testing.T`和`testing.B`都允许通过`Run`方法定义子测试和子基准。使用这样的方式，可以创建多层次的测试代码，同时可以共享初始化与收尾的代码。

子测试的测试多为`当前测试名/子测试名`

以上的测试可以使用以下命令执行：

```text
$ go test -v  app/testing/fibonacci
=== RUN   TestFibonacci
=== RUN   TestFibonacci/negative
=== RUN   TestFibonacci/value_one
=== RUN   TestFibonacci/value_5_million
--- PASS: TestFibonacci (0.00s)
    --- PASS: TestFibonacci/negative (0.00s)
    --- PASS: TestFibonacci/value_one (0.00s)
    --- PASS: TestFibonacci/value_5_million (0.00s)
=== RUN   ExampleFibonacci
--- PASS: ExampleFibonacci (0.00s)
PASS
ok      app/testing/fibonacci   (cached)

$ go test -v -run TestFibonacci/^v app/testing/fibonacci
=== RUN   TestFibonacci
=== RUN   TestFibonacci/value_one
=== RUN   TestFibonacci/value_5_million
--- PASS: TestFibonacci (0.00s)
    --- PASS: TestFibonacci/value_one (0.00s)
    --- PASS: TestFibonacci/value_5_million (0.00s)
PASS
ok      app/testing/fibonacci   0.049s
```

## TestMain函数 

有些时间，测试有一些初始化需要在主线程运行，这时可以使用TestMain函数

```go
func TestMain(m *testing.M)
```

在一个包内只能有一个TestMain方法。会在测试方法运行前调用。在实际的使用中，可以在`m.Run()`方法前编写初始化的代码。TestMain方法退出时，建议以`os.Exit(m.Run())`的形式退出。

```go
func TestMain(m *testing.M) {
	fmt.Println("Test Main")
	os.Exit(m.Run())
}
```

## 并行测试

有的时候，测试用例希望被测试代码并行运行，以验证被测试代码能否在多线程环境下正常运行。

**示例**

```text
mkdir echo
touch echo/echo.go
touch echo/echo_test.go
```

**echo/echo.go**

```go
package echo

func Echo(arg string) string {
	return arg
}
```

**echo/echo_test.go**

```go
package echo

import (
	"fmt"
	"testing"
)

func TestEchoA(t *testing.T) {
	for i := 0; i < 2; i++ {
		got := Echo("A")
		fmt.Println(got)
	}
}

func TestEchoB(t *testing.T) {
	for i := 0; i < 2; i++ {
		got := Echo("B")
		fmt.Println(got)
	}
}

func TestEchoC(t *testing.T) {
	for i := 0; i < 2; i++ {
		got := Echo("C")
		fmt.Println(got)
	}
}

func TestEchoD(t *testing.T) {
	for i := 0; i < 2; i++ {
		got := Echo("D")
		fmt.Println(got)
	}
}
```

运行测试结果如下：

```text
$ go test -v app/testing/echo
=== RUN   TestEchoA
A
A
--- PASS: TestEchoA (0.00s)
=== RUN   TestEchoB
B
B
--- PASS: TestEchoB (0.00s)
=== RUN   TestEchoC
C
C
--- PASS: TestEchoC (0.00s)
=== RUN   TestEchoD
D
D
--- PASS: TestEchoD (0.00s)
PASS
ok      app/testing/echo        0.069s
```

现在修改测试代码，在`TestEchoA`和`TestEchoB`的入口处添加`t.Parallel()`。

```go
package echo

import (
	"fmt"
	"testing"
)

func TestEchoA(t *testing.T) {
	t.Parallel()
	for i := 0; i < 2; i++ {
		got := Echo("A")
		fmt.Println(got)
	}
}

func TestEchoB(t *testing.T) {
	t.Parallel()
	for i := 0; i < 2; i++ {
		got := Echo("B")
		fmt.Println(got)
	}
}

func TestEchoC(t *testing.T) {
	for i := 0; i < 2; i++ {
		got := Echo("C")
		fmt.Println(got)
	}
}

func TestEchoD(t *testing.T) {
	for i := 0; i < 2; i++ {
		got := Echo("D")
		fmt.Println(got)
	}
}
```

再次运行，输出如下

```text
$ go test -v app/testing/echo
=== RUN   TestEchoA
=== PAUSE TestEchoA
=== RUN   TestEchoB
=== PAUSE TestEchoB
=== RUN   TestEchoC
C
C
--- PASS: TestEchoC (0.00s)
=== RUN   TestEchoD
D
D
--- PASS: TestEchoD (0.00s)
=== CONT  TestEchoA
A
A
=== CONT  TestEchoB
B
B
--- PASS: TestEchoB (0.00s)
--- PASS: TestEchoA (0.00s)
PASS
ok      app/testing/echo        0.060s
```

第二次运行测试时，`TestEchoC`并不是在`TestEchoA`和`TestEchoB`运行完后再执行的，而是`TestEchoA`和`TestEchoB`与其它测试用例并行执行。


在基准测试中，可以使用`b.RunParallel`函数并行执行基准测试。

**示例**

改造`BenchmarkFibonacci`基准测试代码如下

```go
func BenchmarkFibonacci(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			Fibonacci(5000000)
		}
	})
}
```

运行效果

```text
$ go test -bench . -run ^$ -cpu 1,2,3,4,5,6,7,8  ./fibonacci
goos: windows
goarch: amd64
pkg: app/testing/fibonacci
BenchmarkFibonacci       3000000               494 ns/op
BenchmarkFibonacci-2    10000000               206 ns/op
BenchmarkFibonacci-3    10000000               171 ns/op
BenchmarkFibonacci-4    10000000               159 ns/op
BenchmarkFibonacci-5    10000000               167 ns/op
BenchmarkFibonacci-6    10000000               219 ns/op
BenchmarkFibonacci-7    10000000               245 ns/op
BenchmarkFibonacci-8     5000000               273 ns/op
PASS
ok      app/testing/fibonacci   16.639s

```

执行并行基准测试时，可以使用`-cpu n,m,...` 来指定不同的运行时`GOMAXPROCS`。默认只执行当前`GOMAXPROCS`

##  单元测试覆盖率

`go test` 命令的`-cover`标记指定输出测试用例的测试覆盖率

```text
$ go test -cover app/testing/fibonacci
ok      app/testing/fibonacci   0.054s  coverage: 100.0% of statements
```

另一种更友好的方式是以profile的形式保存测试覆盖率，并使用`go tool`展示详细的测试用例覆盖信息。

```text
$ go test -coverprofile fibonacci_cover.out app/testing/fibonacci
ok      app/testing/fibonacci   0.051s  coverage: 100.0% of statements

$ go tool cover -html fibonacci_cover.out

```

此命令执行后，会打开默认浏览器并展示哪些代码被覆盖到了，哪些没有被覆盖到。

![覆盖率](/images/go/testing/cover.png)

## 运行时数据采样

在运行基准测试时，如果某部分代码运行效率较低，需要分析低效的原因。此时，可以在基准测试，对运行时的内存，CPU，阻塞，锁状态进行采样。

示例

```text
$ mkdir profile
$ go test -bench . -run ^$ -blockprofile block.out -cpuprofile cpu.out -memprofile mem.out -mutexprofile mutex.out  -trace trace.out -outputdir ../profile ./fibonacci
$ ls profile/
block.out  cover.out  cpu.out  mem.out  mutex.out  trace.out
```

参数说明：

* `-blockprofile block.out`    阻塞采样输出到block.out
* `-cpuprofile cpu.out`    CPU采样数据输出到cpu.out
* `-memprofile mem.out`    内存采样数据输出到mem.out
* `-mutexprofile mutex.out`    加锁与释放锁的采样数据输出到mutex.out
* `-trace trace.out`    执行追踪数据输出到trace.out
* `-outputdir ../profile`    指定采样数据存在目录，这个参数是相对于基准测试的包的目录来的，**不是指当前目录**。


有了这些采样文件，就可以使用`go tool`来进行性能分析了，当然对哪些数据进行采样，应该跟据情况设置，并不需要对所有数据进行采样，毕竟采样也是有性能损耗的。

## 参考链接

[testing - The Go Programming Language](https://golang.org/pkg/testing/)

### 命令

```text
go help testflag
```
