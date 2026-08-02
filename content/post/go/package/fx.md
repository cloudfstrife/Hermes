---
title: "Uber Fx依赖注入框架"
date: 2026-08-02T15:14:06+08:00
lastmod: 2026-08-02T15:14:06+08:00

categories:
  - go
tags:
  - go
  - fx
keywords:
  - go
  - fx

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

[Fx](GitHub - uber-go/fx: A dependency injection based application framework for Go.)是Uber开发并开源的Go语言模块组合框架，Fx提供了模块化、可插拔、可组合的方式来构建Go应用。与 [wire](GitHub - google/wire: Compile-time Dependency Injection for Go)生成代码的方式不同的是：Fx通过反射创建对象和解析依赖关系。

<!--more-->

## 基本使用

本文使用一个最基本的示例程序来展示 Fx 的最基本的功能，更复杂的使用示例参见官方文档：[Get started with Fx - Fx](https://uber-go.github.io/fx/get-started/index.html)

### Step.1 创建Go语言项目

```bash
go mod init app.tech/proj/fx-testing
```

### Step.2 创建 main.go 

```
cat << EOF | tee main.go
package main

import (
    "context"
    "fmt"

    "go.uber.org/fx"
)

func main() {
    fx.New(
        fx.Provide(NewPerson),
        fx.Decorate(DecoratePerson),
        fx.Invoke(ShowPerson),
    ).Run()
}

type Person struct {
    Name string
}

func NewPerson() (*Person, error) {
    return &Person{
        Name: "name",
    }, nil
}

func DecoratePerson(person *Person) (*Person, error) {
    person.Name = person.Name + " " + "surname"
    return person, nil
}

func ShowPerson(l fx.Lifecycle, person *Person) {
    l.Append(fx.Hook{
        OnStart: func(context.Context) error {
            fmt.Println("start")
            return nil
        },
        OnStop: func(ctx context.Context) error {
            fmt.Println("stop")
            return nil
        },
    })

    fmt.Printf("%s\n", person.Name)
}
EOF
```

### Step.3 添加依赖 

```bash
go get go.uber.org/fx@latest
```

### Step.4 构建与运行

```bash
go run .
```

输出如下：

```text
[Fx] PROVIDE    fx.Lifecycle <= go.uber.org/fx.New.func1()
[Fx] PROVIDE    fx.Shutdowner <= go.uber.org/fx.(*App).shutdowner-fm()
[Fx] PROVIDE    fx.DotGraph <= go.uber.org/fx.(*App).dotGraph-fm()
[Fx] PROVIDE    *main.Person <= main.NewPerson()
[Fx] DECORATE   *main.Person <= main.DecoratePerson()
[Fx] INVOKE             main.ShowPerson()
[Fx] BEFORE RUN provide: go.uber.org/fx.New.func1()
[Fx] RUN        provide: go.uber.org/fx.New.func1() in 54.917µs
[Fx] BEFORE RUN provide: main.NewPerson()
[Fx] RUN        provide: main.NewPerson() in 168.583µs
[Fx] BEFORE RUN decorate: main.DecoratePerson()
[Fx] RUN        decorate: main.DecoratePerson() in 3.375µs
name surname
[Fx] HOOK OnStart               main.ShowPerson.func1() executing (caller: main.ShowPerson)
start
[Fx] HOOK OnStart               main.ShowPerson.func1() called by main.ShowPerson ran successfully in 8.042µs
[Fx] RUNNING
```
按下 `Ctrl+C` 退出程序，输出如下：

```text
......
[Fx] RUNNING
^C[Fx] INTERRUPT
[Fx] HOOK OnStop                main.ShowPerson.func2() executing (caller: main.ShowPerson)
stop
[Fx] HOOK OnStop                main.ShowPerson.func2() called by main.ShowPerson ran successfully in 123.708µs
```

### 解释

以上的代码通过不带任何参数地调用 `fx.New` 创建了一个空的 Fx 应用程序。随后使用 `App.Run` 方法运行该应用程序。该方法会阻塞直到接收到停止信号，然后在退出前执行必要的清理操作。

Fx 主要面向长期运行的服务器应用程序；此类应用程序通常会在需要关闭时从部署系统接收一个信号。通常情况下，应用程序会向 `fx.New` 传递参数来配置其组件。

## 生命周期

Fx 应用程序的生命周期包含两个高层次阶段：初始化和执行。这两个阶段又分别由多个步骤组成。

在初始化过程中，Fx 将：

* 注册传递给 `fx.Provide` 的所有构造函数
* 注册传递给 `fx.Decorate` 的所有装饰器
* 运行传递给 `fx.Invoke` 的所有函数，并在需要时调用构造函数和装饰器

在执行阶段，Fx 将：

* 运行由 `fx.Provide` ， `fx.Decorate` ， `fx.Invoke` 附加到应用程序上的所有启动钩子（Hook）
* 等待停止运行的信号
* 运行附加到应用程序上的所有关闭钩子

![lifecycle](/images/go/fx/life_cycle.svg)

### 生命周期Hook

生命周期Hook允许在应用程序启动或关闭时，安排由 Fx 执行的相关任务。

Fx 提供了两种类型的 Hook：

* Startup hooks，也称为 OnStart 钩子。这些钩子按添加顺序依次执行。
* Shutdown hooks，也称为 OnStop 钩子。这些钩子按与添加顺序相反的顺序依次执行。

通常提供 Startup hooks 的组件也会提供相应的 Shutdown hooks，以释放其在启动时获取的资源。

Fx 运行这两种钩子时会严格执行超时限制。因此 Hook 不能阻塞或者同步执行长时间运行的任务，如果存在这种长时间运行的任务，应该使用 goroutine ，在关闭任务中确保这些 goroutine 正确退出。

## Container

容器是一种负责容纳所有构造函数和值的抽象概念。它是应用程序与 Fx 交互的主要途径。应用程序需要向容器说明自身的需求以及如何执行特定操作，然后由容器来实际运行应用程序。

Fx 并不提供对容器的直接访问。相反需要通过向 fx.New 构造函数提供 [fx.Options](https://pkg.go.dev/go.uber.org/fx#Option) 来指定要在容器上执行的操作。


## fx.Options

### Provide

在使用容器之前，必须先为其提供值。Fx 提供了两种向容器提供值的方法：

* `fx.Provide` 用于注册具有构造函数的值。

```go
fx.Provide(
  func(cfg *Config) *Logger { /* ... */ },
)
```

* `fx.Supply` 用于预定义的非接口值。
```go
fx.Provide(
  fx.Supply(&Config{
    Name: "my-app",
  }),
)
```

### Invoke

向容器提供值仅使应用程序能够创建这些值，但此时容器尚未对这些值进行任何操作。

**传递给 `fx.Provide` 的构造函数只有在需要时才会被调用。** 例如，以下代码不会产生任何效果：

```go
fx.New(
  fx.Provide(newHTTPServer), // provides an *http.Server
).Run()
```

应用程序需要告诉容器需要做什么？以及如何做。所以 Fx 为此提供了 `fx.Invoke` 方法

下面的示例中，应用程序需要容器执行启动服务器的调用：

```go
fx.New(
  fx.Provide(newHTTPServer),
  fx.Invoke(startHTTPServer),
).Run()
```

`fx.Invoke` 通常用于根级调用，例如启动服务器或启动定时任务运行。

### Decorate

`fx.Decorate` 用于为 Fx 应用程序注册类型的装饰器函数。`Decorate` 函数允许用户对对象进行增强。它们可以接收零个或多个依赖项（这些依赖项必须通过 `fx.Provide` 提供给应用程序），并返回一个或多个值，这些值可供其他 `fx.Provide` 和 `fx.Invoke` 调用使用。

```go
fx.Decorate(func(log *zap.Logger) *zap.Logger {
  return log.Named("myapp")
})

fx.Invoke(func(log *zap.Logger) {
  log.Info("hello")
  // Output:
  // {"level": "info","logger":"myapp","msg":"hello"}
})
```

### Module

Fx 模块是一种可共享的 Go 库或包，为 Fx 应用程序提供自包含的功能。

#### 编写模块
编写模块的步骤如下：

* 使用 `fx.Module` 调用生成的顶级 Module 变量。

```go
var Module = fx.Module("server",
```
* 使用 `fx.Provide` 添加模块的组件。

```go
var Module = fx.Module("server",
   fx.Provide(
       New,
   ),
```

* 使用 `fx.Invoke` 添加调用（可选）

```go
var Module = fx.Module("server",
   fx.Invoke(startServer),
```

* 使用 `fx.Decorate` 添加装饰器（可选）

```go
var Module = fx.Module("server",
   fx.Decorate(wrapLogger),
```

* 如果希望将构造函数的输出限制在当前模块以及该模块所包含的模块的范围内，可以在提供时添加 `fx.Private`。

```go
var Module = fx.Module("server",
   fx.Provide(
       fx.Private,
       parseConfig,
   ),
```

#### 使用模块

```go
package main

import (
    "context"
    "fmt"

    "go.uber.org/fx"
)

func main() {
    fx.New(
        fx.Provide(NewPerson),
        fx.Decorate(DecoratePerson),
        fx.Invoke(ShowPerson),
        // 
        fx.Module("version",
            fx.Invoke(ShowVersion),
        ),
        fx.NopLogger,
    ).Run()
}

type Person struct {
    Name string
}

func NewPerson() (*Person, error) {
    return &Person{
        Name: "name",
    }, nil
}

func DecoratePerson(person *Person) (*Person, error) {
    person.Name = person.Name + " " + "surname"
    return person, nil
}

func ShowPerson(l fx.Lifecycle, person *Person) {
    l.Append(fx.Hook{
        OnStart: func(context.Context) error {
            fmt.Println("start")
            return nil
        },
        OnStop: func(ctx context.Context) error {
            fmt.Println("stop")
            return nil
        },
    })

    fmt.Printf("%s\n", person.Name)
}

func ShowVersion() {
    fmt.Println("version")
}
```

> 注意这里的 `Invoke` 的执行顺序，上面的代码会先执行 `ShowVersion` ，再执行 `ShowPerson`。

## 

## 其它

### 禁用Fx日志

`fx.NopLogger` 选项可以禁用fx框架的日志输出

```go
fx.New(
    fx.NopLogger,
).Run()
```