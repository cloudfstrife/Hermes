---
title: "Thanos编码风格指南"
date: 2020-04-22T18:56:56+08:00
categories:
- Go
tags:
- Go
- Thanos
keywords:
- Go
- Thanos
---

> 本文翻译了 [Thanos Coding Style Guide](https://github.com/thanos-io/thanos/blob/master/docs/contributing/coding-style-guide.md) 

<!--more-->

# Thanos Coding Style Guide

# Thanos 编码风格指南

本文档详细介绍了我们在Thanos项目中使用的各种语言的官方样式指南。请随便了解一下，并在代码审查过程中参考这个文档。如果代码库中的某些内容与风格指南不匹配，则表示它有疏漏，或者这部分代码是本文档建立之前写的，你可以帮助修复它！(:

通常，我们关心：

* 可读性，低[认知负荷](https://www.dabapps.com/blog/cognitive-load-programming/).
* 可维护性。 避免使用 **惊喜** 代码。
* 在不影响可读性的前提下，考虑关键路径的性能。
* 可测试性。即使这意味着生产代码要修改，像 `timeNow func() time.Time` 这样的代码也要模拟。
* 一致性。如果某些模式重复出现，就没有什么意义了。

一些风格会被Liners强制执行，并包含在其它的单独的小部分中。如果您想在自己的项目中包含一些规则，请参考那些！
对于Thanos开发者，我们建议您阅读相关规则的部分，并在开发过程中实践。目前，其中的一些规则是Liner无法检测的。理想情况下，一切都将是自动化的。(:

## TOC

## 目录

- [Thanos Coding Style Guide](#thanos-coding-style-guide)  
  * [TOC](#toc)  
- [Go](#go)  
  * [Development / Code Review](#development-code-review)  
     + [Reliability](#reliability)  
        - [Defers: Don't Forget to Check Returned Errors](#defers-don-t-forget-to-check-returned-errors)  
        - [Exhaust Readers](#exhaust-readers)  
        - [Avoid Globals](#avoid-globals)  
        - [Never Use Panics](#never-use-panics)  
        - [Avoid Using the `reflect` or `unsafe` Packages](#avoid-using-the-reflect-or-unsafe-packages)  
        - [Avoid variable shadowing](#avoid-variable-shadowing)  
     + [Performance](#performance)  
        - [Pre-allocating Slices and Maps](#pre-allocating-slices-and-maps)  
        - [Reuse arrays](#reuse-arrays)  
     + [Readability](#readability)  
        - [Keep the Interface Narrow; Avoid Shallow Functions](#keep-the-interface-narrow-avoid-shallow-functions)  
        - [Use Named Return Parameters Carefully](#use-named-return-parameters-carefully)  
        - [Clean Defer Only if Function Fails](#clean-defer-only-if-function-fails)  
        - [Explicitly Handled Returned Errors](#explicitly-handled-returned-errors)  
        - [Avoid Defining Variables Used Only Once.](#avoid-defining-variables-used-only-once)  
        - [Only Two Ways of Formatting Functions/Methods](#only-two-ways-of-formatting-functions-methods)  
        - [Control Structure: Prefer early returns and avoid `else`](#control-structure-prefer-early-returns-and-avoid-else)  
        - [Wrap Errors for More Context; Don't Repeat "failed ..." There.](#wrap-errors-for-more-context-don-t-repeat-failed-there)  
        - [Use the Blank Identifier `_`](#use-the-blank-identifier)  
        - [Rules for Log Messages](#rules-for-log-messages)  
        - [Comment Necessary Surprises](#comment-necessary-surprises)  
     + [Testing](#testing)  
        - [Table Tests](#table-tests)  
        - [Tests for Packages / Structs That Involve `time` package.](#tests-for-packages-structs-that-involve-time-package)  
  * [Enforced by Linters](#enforced-by-linters)  
     - [Avoid Prints](#avoid-prints)  
     - [Ensure Prometheus Metric Registration](#ensure-prometheus-metric-registration)  
     - [go vet](#go-vet)  
     - [golangci-lint](#golangci-lint)  
     - [misspell](#misspell)  
     - [Comments Should be Full Sentences](#comments-should-be-full-sentences)  
- [Bash](#bash)  

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

# Go

对于用 [Go语言](https://golang.org/) 编写的代码，我们使用标准的Go风格指南([Effective Go](https://golang.org/doc/effective_go.html),[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)) 附加一些其它的规则，使当前的风格指南比标准风格指南更加严格一点。
这样可以确保在现代分布式系统数据库（如Thanos）中具有更好的一致性，在这些数据库中，可靠性，性能和可维护性至关重要。

<img src="../img/go-in-thanos.jpg" class="img-fluid" alt="Go in Thanos" />

<!--
NOTE: Because of blackfriday bug, we have to change those code snippet to `< highlight go >` hugo shortcodes during `websitepreprocessing.sh` for Thanos website.
-->

## Development / Code Review

## 开发/代码审查

在本节中，我们将研究在开发和代码审查期间应用的标准指南之上的规则。

NOTE: 如果你知道这些规则中的任何部分可以被liner自动检测，请告诉我们！(:

### Reliability

### 可靠性

编码风格不纯粹用于定义什么样的代码是丑陋的代码，什么样的不是。更重要的是为了确保程序在一天24小时的生产环境中可靠运行，不会引起事故。
以下规则描述了一些我们经常在Go社区中看到的不健康的模式，这些可以被视为错误，或者会大幅增加引入错误的可能。

#### Defers: Don't Forget to Check Returned Errors

#### Defers: 别忘记检查返回错误

我们很容易忽略 `defer` 调用 `Close` 方法返回的错误。

```go
f, err := os.Open(...)
if err != nil {
    // handle..
}
defer f.Close() // What if an error occurs here?

// Write something to file... etc.
```

这样未经检查的错误可能会导致重大错误。上面的例子：`*os.File` `Close`方法负责刷新流到文件，因此，如果此时发生错误，**写入可能会中止！** 😱

检查所有错误！为了保持一致而专注。 可以使用 [runutil](https://pkg.go.dev/github.com/thanos-io/thanos@v0.11.0/pkg/runutil?tab=doc) helper包, 例如:

```go
// Use `CloseWithErrCapture` if you want to close and fail the function or
// method on a `f.Close` error (make sure thr `error` return argument is
// named as `err`). If the error is already present, `CloseWithErrCapture`
// will append (not wrap) the `f.Close` error if any.
defer runutil.CloseWithErrCapture(&err, f, "close file")

// Use `CloseWithLogOnErr` if you want to close and log error on `Warn`
// level on a `f.Close` error.
defer runutil.CloseWithLogOnErr(logger, f, "close file")
```

**避免 🔥**

```go
func writeToFile(...) error {
    f, err := os.Open(...)
    if err != nil {
        return err
    }
    defer f.Close() // What if an error occurs here?

    // Write something to file...
    return nil
}
```

**更好 🤓**

```go
func writeToFile(...) (err error) {
    f, err := os.Open(...)
    if err != nil {
        return err
    }
    // Now all is handled well.
    defer runutil.CloseWithErrCapture(&err, f, "close file")

    // Write something to file...
    return nil
}
```

#### Exhaust Readers

#### Exhaust读取器

最常见的错误之一就是忘记关闭或完全读取HTTP请求和响应的正文，尤其是在出错时。如果您读取此类结构的正文，可以使用 [runutil](https://pkg.go.dev/github.com/thanos-io/thanos@v0.11.0/pkg/runutil?tab=doc) ，如下：

```go
defer runutil.ExhaustCloseWithLogOnErr(logger, resp.Body, "close response")
```

**避免 🔥**

```go
resp, err := http.Get("http://example.com/")
if err != nil {
    // handle...
}
defer runutil.CloseWithLogOnErr(logger, resp.Body, "close response")

scanner := bufio.NewScanner(resp.Body)
// If any error happens and we return in the middle of scanning
// body, we can end up with unread buffer, which
// will use memory and hold TCP connection!
for scanner.Scan() {
```

**更好 🤓**

```go
resp, err := http.Get("http://example.com/")
if err != nil {
    // handle...
}
defer runutil.ExhaustCloseWithLogOnErr(logger, resp.Body, "close response")

scanner := bufio.NewScanner(resp.Body)
// If any error happens and we return in the middle of scanning body,
// defer will handle all well.
for scanner.Scan() {
```

#### Avoid Globals

#### 避免使用全局变量

除`const`外，不允许使用其他全局变量。这也意味着不需要`init`函数。

#### Never Use Panics

#### 不要使用Panics

永远不要使用 Panics，如果依赖的包中使用了 Panics，使用 [recover](https://golang.org/doc/effective_go.html#recover)。另外，请考虑去除这样的依赖。 🙈

#### Avoid Using the `reflect` or `unsafe` Packages

#### 避免使用 `reflect` 和 `unsafe` 包

仅用于非常特殊的，或者非常紧急的情况。特别是 `reflect` 非常慢。对于测试代码，可以使用反射。

#### Avoid variable shadowing

#### 避免阴影变量

阴影变量是指在较小的作用域中使用同名的变量。这是非常危险的，因为它会导致许多意外情况。调试此类问题非常困难，它们可能出现在代码的无关部分中。And what's broken is tiny `:` or lack of it.

**避免 🔥**

```go
    var client ClientInterface
    if clientTypeASpecified {
        // Ups - typo, should be =`
        client, err := clienttypea.NewClient(...)
        if err != nil {
            // handle err
        }
        level.Info(logger).Log("msg", "created client", "type", client.Type)
    } else {
        // Ups - typo, should be =`
         client, err := clienttypea.NewClient(...)
         level.Info(logger).Log("msg", "noop client will be used", "type", client.Type)
    }

    // In some further deeper part of the code...
    resp, err := client.Call(....) // nil pointer panic!
```

**更好 🤓**

```go
    var client ClientInterface = NewNoop(...)
    if clientTypeASpecified {
        c, err := clienttypea.NewClient(...)
        if err != nil {
            // handle err
        }
        client = c
    }
    level.Info(logger).Log("msg", "created client", "type", c.Type)

    resp, err := client.Call(....)
```

这也是为什么我们建议在可能小的作用域下处理错误的原因：

```go
    if err := doSomething; err != nil {
        // handle err
    }
```

虽然尚未配置，但我们可能会考虑在未来不允许使用 [`golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow`](https://godoc.org/golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow)。甚至Go2的提案也有说明 [disabling this in the language itself, but was rejected](https://github.com/golang/go/issues/21114):

与该问题类似的是阴影包名。尽管危险性较小，但它可能会引起类似的问题，所以请尽可能避免使用阴影包名。

### Performance

### 性能

毕竟，Thanos系统是一个在人类可接受的响应时间内搜索上TB数据的数据库。这意味着需要在我们的代码中添加一些其他模式。对于这些模式，我们尽量不牺牲可读性，并仅应用于关键代码路径。

**请记住要衡量结果。** Go语言的性能取决于许多隐藏的事物和调整，所以好的微基准测试，遵循真正的系统负载测试，在大多数情况下，需要了解优化是否有意义。

#### Pre-allocating Slices and Maps

#### 预分配Slices和Maps

尽量预分配Slices和Maps。如果你知道存放数据的数量，就要利用这些信息！这能明显改善此类代码的延迟。这被视为微优化，但是这是一个好的模式，因为它很简单。在性能方面，这个模式仅与具有大数组的关键代码路径有关。

NOTE: 这是因为，从简单的视角来看，Go语言运行时会分配2倍于当前Slices和Maps的空间（译注: slice 扩容时，容量小于1024是翻倍，大于1024是1.25倍），所以，如果元素的数量上百万时，Go分在执行`append`期间进行大量的内存分配。

**避免 🔥**

```go
func copyIntoSliceAndMap(biggy []string) (a []string, b map[string]struct{})
    b = map[string]struct{}{}

    for _, item := range biggy {
        a = append(a, item)
        b[item] = struct{}
    }
}
```

**更好 🤓**

```go
func copyIntoSliceAndMap(biggy []string) (a []string, b map[string]struct{})
    b = make(map[string]struct{}, len(biggy))
    a = make([]string, len(biggy))

    // Copy will not even work without pre-allocation.
    copy(a, biggy)
    for _, item := range biggy {
        b[item] = struct{}
    }
}
```

#### Reuse arrays

#### 重用数组

为了扩展上述观点，在某些情况下，您不需要一直在内存中分配新的空间。如果您对slices依次重复执行某些操作，每次迭代就释放数组，请合理地为它们重用基础数组。这可以为关键路径带来巨大的性能提升。但是，不幸的是当前无法将底层数组重用于Map。

NOTE: 为什么不能只在每个的迭代中重复分配和次释放呢？ Go语言应该知道有空间并重用它吧？ (: 好吧，事实并没有那么简单。长话短说(TL;DR)就是Go语言的垃圾回收器是定期运行或在某些情况下运行（大堆），但绝对不是在循环的每次迭代中（那会非常慢），详情请阅读 [here](https://about.sourcegraph.com/go/gophercon-2018-allocator-wrestling).

**避免 🔥**

```go
var messages []string{}
for _, msg := range recv {
    messages = append(messages, msg)

    if len(messages) > maxMessageLen {
        marshalAndSend(messages)
        // This creates new array. Previous array
        // will be garbage collected only after
        // some time (seconds), which
        // can create enormous memory pressure.
        messages = []string{}
    }
}
```

**更好 🤓**

```go
var messages []string{}
for _, msg := range recv {
    messages = append(messages, msg)

    if len(messages) > maxMessageLen {
        marshalAndSend(messages)
        // Instead of new array, reuse
        // the same, with the same capacity,
        // just length equals to zero.
        messages = messages[:0]
    }
}
```


### Readability

### 可读性

这部分是所有Gophers喜欢的 ❤️ 如何使代码更具可读性？

对于Thanos团队而言，可读性是指以一种不会令代码读者感到惊讶的方式进行编程。所有的细节和不一致之处可能会分散读者的注意力或引起误解，因此每个字符或换行符都可能很重要。
这就是为什么我们要在每个 Pull Requests 的审查上花更多的时间，尤其是在开始时！为确保我们可以快速了解，扩展和修复系统问题。

#### Keep the Interface Narrow; Avoid Shallow Functions

#### Interface尽量小；避免Shallow函数

这点更关乎API设计而不是编码，但是即使在小的编码决策过程中，它也很重要。

* 简单的接口 ( 通常比较小 ) 。这可能意味着接口中的方法更小，函数签名更简单，接口中方法的更少。如果可以，尝试根据功能对接口进行分组，每个接口最多暴露1-3个方法。

**避免 🔥**

```go
// Compactor aka: The Big Boy. Such big interface is really useless ):
type Compactor interface {
    Compact(ctx context.Context) error
    FetchMeta(ctx context.Context) (metas map[ulid.ULID]*metadata.Meta, partial map[ulid.ULID]error, err error)
    UpdateOnMetaChange(func([]metadata.Meta, error))
    SyncMetas(ctx context.Context) error
    Groups() (res []*Group, err error)
    GarbageCollect(ctx context.Context) error
    ApplyRetentionPolicyByResolution(ctx context.Context, logger log.Logger, bkt objstore.Bucket) error
    BestEffortCleanAbortedPartialUploads(ctx context.Context, bkt objstore.Bucket)
    DeleteMarkedBlocks(ctx context.Context) error
    Downsample(ctx context.Context, logger log.Logger, metrics *DownsampleMetrics, bkt objstore.Bucket) error
}
```

**更好 🤓**

```go
// Smaller interfaces with a smaller number of arguments allow functional grouping, clean composition and clear testability.
type Compactor interface {
    Compact(ctx context.Context) error

}

type Downsampler interface {
    Downsample(ctx context.Context) error
}

type MetaFetcher interface {
    Fetch(ctx context.Context) (metas map[ulid.ULID]*metadata.Meta, partial map[ulid.ULID]error, err error)
    UpdateOnChange(func([]metadata.Meta, error))
}

type Syncer interface {
    SyncMetas(ctx context.Context) error
    Groups() (res []*Group, err error)
    GarbageCollect(ctx context.Context) error
}

type RetentionKeeper interface {
    Apply(ctx context.Context) error
}

type Cleaner interface {
    DeleteMarkedBlocks(ctx context.Context) error
    BestEffortCleanAbortedPartialUploads(ctx context.Context)
}
```

* 最好对用户隐藏不必要的复杂性。这意味着shallow函数会引入更多理解函数名称，寻找函数实现以理解函数的认知负担。将这几行直接内联到调用方可能更容易理解。

**避免 🔥**

```go
    // Some code...
    s.doSomethingAndHandleError()

    // Some code...
}

func (s *myStruct) doSomethingAndHandleError() {
    if err := doSomething; err != nil {
        level.Error(s.logger).Log("msg" "failed to do something; sorry", "err", err)
    }
}
```

**更好 🤓**


```go
    // Some code...
    if err := doSomething; err != nil {
        level.Error(s.logger).Log("msg" "failed to do something; sorry", "err", err)
    }

    // Some code...
}
```

</td></tr>
</tbody></table>

这与 `尽量找一种，最好是唯一一种明确的解决方案` 和 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 规则有一点关联. 如果您做某事的方式多于一种，这意味着您将拥有更宽泛的Interface，那么就有更大的可能引入错误，歧义和维护负担。

**避免 🔥**

```go
// We have here SIX potential how caller can get an ID. Can you find all of them?

type Block struct {
    // Things...
    ID ulid.ULID

    mtx sync.Mutex
}

func (b *Block) Lock() {  b.mtx.Lock() }

func (b *Block) Unlock() {  b.mtx.Unlock() }

func (b *Block) ID() ulid.ULID {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    return b.ID
}

func (b *Block) IDNoLock() ulid.ULID {  return b.ID }
```

**更好 🤓**

```go
type Block struct {
    // Things...

    id ulid.ULID
    mtx sync.Mutex
}

func (b *Block) ID() ulid.ULID {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    return b.id
}
```

#### Use Named Return Parameters Carefully

#### 慎用命名返回参数

如果类型不能提供足够的函数返回值信息时，可以使用命名返回参数，另一种场景是当你想定义一个变量时，例如slice

**重要提示**：不要在已命名返回值的函数中直接使用`return`。这可以编译，但是会使返回值隐藏起来，更容易出现意外情况。

#### Clean Defer Only if Function Fails

#### 仅在函数失败时，清理Defer 

有一种方法，可以牺牲延迟，以便正确关闭所有错误。重复会在更改代码时更容易出错并忘记修改某些东西，因此可以像下面一样执行错误处理：

**避免 🔥**

```go
func OpenSomeFileAndDoSomeStuff() (*os.File, error) {
    f, err := os.OpenFile("file.txt", os.O_RDONLY, 0)
    if err != nil {
        return nil, err
    }

    if err := doStuff1(); err != nil {
        runutil.CloseWithErrCapture(&err, f, "close file")
        return nil, err
    }
    if err := doStuff2(); err != nil {
        runutil.CloseWithErrCapture(&err, f, "close file")
        return nil, err
    }
    if err := doStuff232241(); err != nil {
        // Ups.. forgot to close file here.
        return nil, err
    }
    return f, nil
}
```

**更好 🤓**

```go
func OpenSomeFileAndDoSomeStuff() (f *os.File, err error) {
    f, err = os.OpenFile("file.txt", os.O_RDONLY, 0)
    if err != nil {
        return nil, err
    }
    defer func() {
        if err != nil {
             runutil.CloseWithErrCapture(&err, f, "close file")
        }
    }

    if err := doStuff1(); err != nil {
        return nil, err
    }
    if err := doStuff2(); err != nil {
        return nil, err
    }
    if err := doStuff232241(); err != nil {
        return nil, err
    }
    return f, nil
}
```

#### Explicitly Handled Returned Errors

#### 明确处理返回的错误

处理任何返回的错误。不是说不能“忽略”出于某种原因产生错误，比如：如果知道实现不会返回任何有意义的东西，可以忽略该错误，但是要明确地这样做：

**避免 🔥**

```go
someMethodThatReturnsError(...)
```

**更好 🤓**


```go
_ = someMethodThatReturnsError(...)
```

例外: 著名的 `level.Debug|Warn` 和 `fmt.Fprint*`

#### Avoid Defining Variables Used Only Once.

#### 避免定义仅使用一次的变量

定义变量用于创建一个很大的东西，这很诱人，如果仅使用一次，则避免定义此类变量。当你创建一个变量时，读者会认为这个变量有其它的用途，每次检查，最终意识到它只使用一次可能会很烦人。

**避免 🔥**

```go
    someConfig := a.GetConfig()
    address124 := someConfig.Addresses[124]
    addressStr := fmt.Sprintf("%s:%d", address124.Host, address124.Port)

    c := &MyType{HostPort: addressStr, SomeOther: thing}
    return c
```

**更好 🤓**

```go
    // This variable is required for potentially consistent results. It is used twice.
    someConfig := a.FetchConfig()
    return &MyType{
        HostPort:  fmt.Sprintf("%s:%d", someConfig.Addresses[124].Host, someConfig.Addresses[124].Port),
        SomeOther: thing,
    }
```

#### Only Two Ways of Formatting Functions/Methods

#### 仅两种格式化函数/方法的方式

更好把带参数的函数/方法定义在一行中。如果太宽，请将每个参数放在新行上。

**避免 🔥**

```go
func function(argument1 int, argument2 string,
    argument3 time.Duration, argument4 someType,
    argument5 float64, argument6 time.Time,
) (ret int, err error) {
```

**更好 🤓**

```go
func function(
    argument1 int,
    argument2 string,
    argument3 time.Duration,
    argument4 someType,
    argument5 float64,
    argument6 time.Time,
) (ret int, err error)
```

这一点同时适用于调用和定义方法/函数。

NOTE: 有一种例外是可变参数（例如`...string`）必须成对填写时。例如：

```go
level.Info(logger).Log(
    "msg", "found something epic during compaction; this looks amazing",
    "compNumber", compNumber,
    "block", id,
    "elapsed", timeElapsed,
)
```

#### Control Structure: Prefer early returns and avoid `else`

#### 控制结构：尽早return 避免使用 else

在大多数情况下，不需要`else`。通常，您可以使用`continue`，`break`或`return`来结束`if`块。这样可以减少缩进量，代码更易读。

**避免 🔥**

```go
for _, elem := range elems {
    if a == 1 {
        something[i] = "yes"
    } else
        something[i] = "no"
    }
}
```

**更好 🤓**

```go
for _, elem := range elems {
    if a == 1 {
        something[i] = "yes"
        continue
    }
    something[i] = "no"
}
```

#### Wrap Errors for More Context; Don't Repeat "failed ..." There.

#### 包装 Errors 以获得更多上下文信息; 不要重复 "failed ..."

我们使用 [`pkg/errors`](https://github.com/pkg/errors) 包用于处理`errors`，相比于标准库的 `fmt.Errorf` + `%w`我们更喜欢用它,
`errors.Wrap`是显性的。使用标准库很容易 偶然的将`%w`替换为`%v`或在字符串中添加额外的不一致字符。

在发生错误时使用 [`pkg/errors.Wrap`](https://github.com/pkg/errors) 将错误及当时的上下文包装起来。强烈建议使用`errors.Wrapf`包装变量及上下文，例如：文件名，ID，及失败的操作等.

NOTE: 不要使用 `failed ... ` 或者 `error occurred while...`这样的前缀包装消息。只需描述发生Error产生时我们想要做什么，这样的前缀只是噪音。我们正在包装Error，因此很明显产生了一些Error，对吧？(: 提高可读性，并避免这种情况


**避免 🔥**

```go
if err != nil {
    return fmt.Errorf("error while reading from file %s: %w", f.Name, err)
}
```

**更好 🤓**

```go
if err != nil {
    return errors.Wrapf(err, "read file %s", f.Name)
}
```

#### Use the Blank Identifier `_`

#### 使用空白标识符 `_`

空白标识符对于标记未使用的变量非常有用。考虑下面的情况：

```go
// We don't need the second return parameter.
// Let's use the blank identifier instead.
a, _, err := function1(...)
if err != nil {
    // handle err
}
```

```go
// We don't need to use this variable, we
// just want to make sure TypeA implements InterfaceA.
var _ InterfaceA = TypeA
```

```go
// We don't use context argument; let's use the blank
// identifier to make it clear.
func (t *Type) SomeMethod(_ context.Context, abc int) error {
```

#### Rules for Log Messages

#### 日志消息规则

我们在Thanos中使用 [go-kit logger](https://github.com/go-kit/kit/tree/master/log) 。这意味着我们期望日志行具有结构性。结构意味着应该将变量作为单独的字段传递，而不是向消息中组合变量。注意，Thanos的日志全部应该 `小写` （可读性和一致性），所有的结构体键应该是 `驼峰式` 的。建议键名尽量简短一致。例如，如果我们一直使用 `block` 代表 BlockID，那么其他单个日志消息中不要使用 `id` 代表它。

**避免 🔥**

```go
level.Info(logger).Log("msg", fmt.Sprintf("Found something epic during compaction number %v. This looks amazing.", compactionNumber),
 "block_id", id, "elapsed-time", timeElapsed)
```

**更好 🤓**

```go
level.Info(logger).Log("msg", "found something epic during compaction; this looks amazing", "compNumber", compNumber,
"block", id, "elapsed", timeElapsed)
```

此外，在使用不同的日志级别时，我们建议使用某些规则：

* level.Info: 应始终具有 `msg` 字段。仅应用于预计不太经常发生的重要事件。
* level.Debug: 应始终具有 `msg` 字段。 它可能产生很多日志，但也不应该无处不在，仅当想真正深入研究某些领域的某些问题时才使用它。
* level.Warn: 应该至少具有 `msg` 或者 `err` 中的一个或者两个。他们应该警告需要调查的可疑事件但处理过程可以处理它们。 尽量尝试描述当前是 *如何* 处理这个警告的，例如：`value will be skipped`
* level.Error: 应该至少具有 `msg` 或者 `err` 中的一个或者两个。仅将其用于紧急事件。

#### Comment Necessary Surprises

#### 必须对可能令用户感到疑惑的代码进行注释

注释并不是好的选择，很可能会很快过时，忘记更新，编译器也不是会报错。因此，仅在必要时使用注释。**并且必须对可能令用户感到疑惑的代码进行注释。**有时，复杂性是必要的，例如为了提高性能。在这种情况下，注释一下为什么需要这种优化。
如果功能只是暂时完成了，可以添加`TODO(<github name>): <something, with GitHub issue link ideally>`.

### Testing

### 测试

#### Table Tests

#### 表测试

为了提高可读性，使用表驱动的测试 [t.Run](https://blog.golang.org/subtests) 。易于阅读，并允许为每个测试用例添加清晰的描述，增加或调整测试用例也更加容易。

**避免 🔥**

```go
host, port, err := net.SplitHostPort("1.2.3.4:1234")
testutil.Ok(t, err)
testutil.Equals(t, "1.2.3.4", host)
testutil.Equals(t, "1234", port)

host, port, err = net.SplitHostPort("1.2.3.4:something")
testutil.Ok(t, err)
testutil.Equals(t, "1.2.3.4", host)
testutil.Equals(t, "http", port)

host, port, err = net.SplitHostPort(":1234")
testutil.Ok(t, err)
testutil.Equals(t, "", host)
testutil.Equals(t, "1234", port)

host, port, err = net.SplitHostPort("yolo")
testutil.NotOk(t, err)
```

**更好 🤓**

```go
for _, tcase := range []struct{
    name string

    input     string

    expectedHost string
    expectedPort string
    expectedErr error
}{
    {
        name: "host and port",

        input:     "1.2.3.4:1234",
        expectedHost: "1.2.3.4",
        expectedPort: "1234",
    },
    {
        name: "host and named port",

        input:     "1.2.3.4:something",
        expectedHost: "1.2.3.4",
        expectedPort: "something",
    },
    {
        name: "just port",

        input:     ":1234",
        expectedHost: "",
        expectedPort: "1234",
    },
    {
        name: "not valid hostport",

        input:     "yolo",
        expectedErr: errors.New("<exact error>")
    },
}{
    t.Run(tcase.name, func(t *testing.T) {
        host, port, err := net.SplitHostPort(tcase.input)
        if tcase.expectedErr != nil {
            testutil.NotOk(t, err)
            testutil.Equals(t, tcase.expectedErr, err)
            return
        }
        testutil.Ok(t, err)
        testutil.Equals(t, tcase.expectedHost, host)
        testutil.Equals(t, tcase.expectedPort, port)
    })
}
```

#### Tests for Packages / Structs That Involve `time` package.

#### 测试涉及`time`包的 Packages / Structs 

避免基于实时的单元测试。尽量mock 结构体诸如 `timeNow func() time.Time` 的时间，对于生产代码，你可以使用 `time.Now` 初始化字段，对于测试代码, 你可以设置一个自定义时间值。

**避免 🔥**

```go
func (s *SomeType) IsExpired(created time.Time) bool {
    // Code is hardly testable.
    return time.Since(created) >= s.expiryDuration
}
```

**更好 🤓**

```go
func (s *SomeType) IsExpired(created time.Time) bool {
    // s.timeNow is time.Now on production, mocked in tests.
    return created.Add(s.expiryDuration).After(s.timeNow())
}
```

## Enforced by Linters

## 强制执行 Linters

这是我们自动化确保的规则列表。这部分是给那些好奇为什么要添加这样的Liner规则和想在类似的Go项目中使用Liner的开发者

#### Avoid Prints

#### 避免使用 Prints

永远不要使用 `print`，使用 `go-kit/log.Logger`。

参见 [here](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L311).

#### Ensure Prometheus Metric Registration

#### 确保 Prometheus 指标被注册

增加 Prometheus 指标 (例如`prometheus.Counter`) 到 `registry.MustRegister` 函数非常容易被忽视。为避免这种情况，我们确保所有指标都是通过 `promtest.With(r).New*` 并且我们不允许使用旧的注册方式。
关于这个问题参见： [here](https://github.com/thanos-io/thanos/issues/2102).

参见 [here](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L308).

#### go vet

标准 Go vet 非常严格, 为了最终结果，时刻审查你的Go代码

参见 [here](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L313).

#### golangci-lint

[golangci-lint](https://github.com/golangci/golangci-lint) 是一个神奇的工具，它允许针对您的代码运行Go社区中的一组自定义linter，给它加星，并使用它吧(:


参见
[here](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L315)   
[those linters](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/.golangci.yml#L31) 

#### misspell

Misspell 也很神奇，它在注释和文档中捕获了拼写错误。

目前还没有语法插件 ): (我们期待).


参见 [here](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L317).

#### Comments Should be Full Sentences

#### 注释应该是完整的句子

注释应该是完整的句子，以大写字母开头，以句号结尾

参见 [here](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L194).

# Bash

原则上尽量不使用bash。超过30行的脚本，考虑使用Go实现 [here](https://github.com/thanos-io/thanos/blob/55cb8ca38b3539381dc6a781e637df15c694e50a/scripts/copyright/copyright.go).

如果一定要写, 我们遵循 [Google Shell style guide](https://google.github.io/styleguide/shellguide.html)
