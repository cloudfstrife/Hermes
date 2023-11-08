---
title: "使用logrus输出日志"
date: 2019-03-18T12:16:31+08:00
categories:
- Go
tags:
- Go
- log
- logrus
keywords:
- Go
- log
- logger
---

logrus是go语言的结构化日志程序，完全兼容于标准库log的API。

<!--more-->

>Case-sensitivity
>
> 该组织的名称已更改为小写——并且不会再更改回来，如果由于大小写敏感而导致导入冲突，请使用小写的 `import: github.com/sirupsen/logrus`。

## 示例

```go
package main

import (
  log "github.com/sirupsen/logrus"
)

func main() {
  log.WithFields(log.Fields{
    "animal": "walrus",
  }).Info("A walrus appears")
}
```

因为logrus的API与标准log库的API兼容，所以这里logrus的导入使用log做为别名。如果要升级使用标准日志库的代码到logrus日志库，只需要使用上面的import替换标准库的import就可以了。

## 个性化配置

```go
package main

import (
  "os"
  log "github.com/sirupsen/logrus"
)

func init() {
  // Log as JSON instead of the default ASCII formatter.
  log.SetFormatter(&log.JSONFormatter{})

  // Output to stdout instead of the default stderr
  // Can be any io.Writer, see below for File example
  log.SetOutput(os.Stdout)

  // Only log the warning severity or above.
  log.SetLevel(log.WarnLevel)
}

func main() {
  log.WithFields(log.Fields{
    "animal": "walrus",
    "size":   10,
  }).Info("A group of walrus emerges from the ocean")

  log.WithFields(log.Fields{
    "omg":    true,
    "number": 122,
  }).Warn("The group's number increased tremendously!")

  log.WithFields(log.Fields{
    "omg":    true,
    "number": 100,
  }).Fatal("The ice breaks!")

  // A common pattern is to re-use fields between logging statements by re-using
  // the logrus.Entry returned from WithFields()
  contextLogger := log.WithFields(log.Fields{
    "common": "this is a common field",
    "other": "I also should be logged always",
  })

  contextLogger.Info("I'll be logged with common and other field")
  contextLogger.Info("Me too")
}
```

## 创建不同的日志记录器

```go
package main

import (
  "os"
  "github.com/sirupsen/logrus"
)

// Create a new instance of the logger. You can have any number of instances.
var log = logrus.New()

func main() {
  // The API for setting attributes is a little different than the package level
  // exported logger. See Godoc.
  log.Out = os.Stdout

  // You could set this to any `io.Writer` such as a file
  // file, err := os.OpenFile("logrus.log", os.O_CREATE|os.O_WRONLY, 0666)
  // if err == nil {
  //  log.Out = file
  // } else {
  //  log.Info("Failed to log to file, using default stderr")
  // }

  log.WithFields(logrus.Fields{
    "animal": "walrus",
    "size":   10,
  }).Info("A group of walrus emerges from the ocean")
}
```

## 字段

logrus鼓励使用字段来谨慎的结构化日志消息，而不是使用很长的，不可解析文本做为日志消息。例如：`log.Fatalf("Failed to send event %s to topic %s with key %d")`可以使用如下的方式来记录

```go
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```

这样产生的日志是可解读的结构化日志。

JSONFormat

```json
{"event":"onchange","key":"001","level":"fatal","msg":"Failed to send event","time":"2019-03-18T17:09:46+08:00","topic":"logrus"}
```

TextFormat

```text
time="2019-03-18T17:11:41+08:00" level=fatal msg="Failed to send event" event=onchange key=001 topic=logrus
```
### 默认字段

通常在日志上添加一些公有字段是非常常见的，例如，在网络程序中添加用户的IP地址和requestID，可以定义一个带字段的日志Entity，使用这个日志对象记录日志就不需要每行日志记录语句都写WithFields

```go
requestLogger := log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})
requestLogger.Info("something happened on that request") # will log request_id and user_ip
requestLogger.Warn("something not great happened")
```

### 记录方法名

如果希望日志记录附带调用方法字段，可以使用下面的方法来设置：

```go
log.SetReportCaller(true)
```

> **注意** : 这会添加不小的开销，取决与go版本，1.6与1.7的版本测试中，性能损失20%-40%，可以通过基准测试来验证：

```text
$ go test -bench=.*CallerTracing
```

## Hook

你可以为特定级别的日志添加Hook，例如，在Error,Fatal和Panic时，将错误信息发送到日志追踪服务，将info发送到statsD或者多个日志记录位置。logrus自带Hook支持，可以在init时，添加这些Hook或者自己定义自己的Hook

```go
import (
	log "github.com/sirupsen/logrus" // the package is named "airbrake"
	logrus_syslog "github.com/sirupsen/logrus/hooks/syslog"
	"log/syslog"
)

func init() {

	// Use the Airbrake hook to report errors that have Error severity or above to
	// an exception tracker. You can create custom hooks, see the Hooks section.
	log.AddHook(airbrake.NewHook(123, "xyz", "production"))

	hook, err := logrus_syslog.NewSyslogHook("udp", "localhost:514", syslog.LOG_INFO, "")
	if err != nil {
		log.Error("Unable to connect to local syslog daemon")
	} else {
		log.AddHook(hook)
	}
}
```

目前已经的Hook在[wiki](https://github.com/sirupsen/logrus/wiki/Hooks)中可以找到

## 日志等级

logrus目前支持七级日志： Trace, Debug, Info, Warning, Error, Fatal and Panic.

```go
log.Trace("Something very low level.")
log.Debug("Useful debugging information.")
log.Info("Something noteworthy happened!")
log.Warn("You should probably take a look at this.")
log.Error("Something failed but I'm not quitting.")
// Calls os.Exit(1) after logging
log.Fatal("Bye.")
// Calls panic() after logging
log.Panic("I'm bailing.")
```

可以在日志记录器上设置日志级别，日志记录器只会记录等于或高于设置的日志级别的日志。

```go
// Will log anything that is info or above (warn, error, fatal, panic). Default.
log.SetLevel(log.InfoLevel)
```

除了使用`WithField`和`WithFields`设置的字段外，日志记录器自动添加以下字段：

**time**		&nbsp;&nbsp;日志时间

**msg**			&nbsp;&nbsp;日志消息

**level**		&nbsp;&nbsp;日志等级

## 环境

logrus没有环境的概念，如果应用要在不同环境设置不同的日志输出，则需要额外的处理。例如：程序使用一个字符串标识当前运行环境，你可以使用如下的处理方式：

```go
import (
  log "github.com/sirupsen/logrus"
)

func init() {
  // do something here to set environment depending on an environment variable
  // or command-line flag
  if Environment == "production" {
    log.SetFormatter(&log.JSONFormatter{})
  } else {
    // The TextFormatter is default, you don't actually have to do this.
    log.SetFormatter(&log.TextFormatter{})
  }
}
```

上面的示例中：生产环境使用JSON格式的日志方便使用Splunk或者logstatsh进行日志聚合。其它环境使用普通文本的日志格式，方便阅读。

## 日志格式

logrus内置了两种格式：

* `logrus.TextFormatter` 如果输出目标是TTY则以彩色文本输出，否则以普通文本输出。

> 如果要在非TTY中输出，可以设置`ForceColors`为`true`，如果要在TTY中输出非彩色文本，可以设置`DisableColors`为`true`

```go
log.SetFormatter(&log.TextFormatter{
	DisableColors: true,
})
```

> 彩色输出时，日志级别会被截取成4个字符，如果要禁用截取，可以设置`DisableLevelTruncation`为`true`。

* `logrus.JSONFormatter` 以json格式输出日志字段。

### 第三方日志格式

* [FluentdFormatter](https://github.com/joonix/log) : 将日志格式化成可由Kubernetes及谷歌容器引擎解析的格式。
* [GELF](https://github.com/fabienm/go-logrus-formatters) : 格式化成[GELF1.1规范](http://docs.graylog.org/en/2.4/pages/gelf.html)的格式
* [logstash](https://github.com/bshuster-repo/logrus-logstash-hook) : 格式化成 [Logstash](http://logstash.net) 事件。
* [prefixed](https://github.com/x-cray/logrus-prefixed-formatter) : 显示日志条目源以及其他布局。
* [zalgo](https://github.com/aybabtme/logzalgo)
* [nested-logrus-formatter](https://github.com/antonfisher/nested-logrus-formatter).

也可以通过实现`Formatter`接口的方式实现自定义格式。

```go
type MyJSONFormatter struct {
}

log.SetFormatter(new(MyJSONFormatter))

func (f *MyJSONFormatter) Format(entry *Entry) ([]byte, error) {
  // Note this doesn't include Time, Level and Message which are available on
  // the Entry. Consult `godoc` on information about those fields or read the
  // source of the official loggers.
  serialized, err := json.Marshal(entry.Data)
    if err != nil {
      return nil, fmt.Errorf("Failed to marshal fields to JSON, %v", err)
    }
  return append(serialized, '\n'), nil
}
```

## logger做为io.Writer

logrus可以转换成`io.Writer`，writer做为`io.Pipe`的终端需要手动关闭

```go
w := logger.Writer()
defer w.Close()

srv := http.Server{
    // create a stdlib log.Logger that writes to
    // logrus.Logger.
    ErrorLog: log.New(w, "", 0),
}
```

写入这个writer的每一行都以一般方式打印，使用格式化器和Hook，级别使用info，这样我们可以很容易的重写标准库的logger

```go
logger := logrus.New()
logger.Formatter = &logrus.JSONFormatter{}

// Use logrus for standard log output
// Note that `log` here references stdlib's log
// Not logrus imported under the name `log`.
log.SetOutput(logger.Writer())
```

## 日志滚动

Logrus不提供日志滚动功能，日志滚动功能可由外部程序提供，例如：logrotate，可提供日志压缩 与删除旧日志的功能。这不应该是应用程序级日志程序的特性。

## 工具

| Tool                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Logrus Mate](https://github.com/gogap/logrus_mate)          | Logrus mate是Logrus管理日志记录器的工具，开发者可以通过配置文件初始化日志记录器的级别、Hook和格式化程序，使用不同的配置文件即可生成适用于不同的环境的日志记录器 |
| [Logrus Viper Helper](https://github.com/heirko/go-contrib/tree/master/logrusHelper) | Logrus Viper Helper封装了使用spf13/Viper配置Logrus的Helper，使用Logrus Mate的一些行为简化配置 |

## 测试

Logrus有一个内置用于断言日志消息是否存在的工具。使用测试Hook实现，提供以下功能：

* 使用`test.NewLocal`和`test.NewGlobal`装饰已有的日志记录器可以使用测试Hook

* `test.NewNullLogger`可以用于记录日志消息（不输出）

```go
import(
  "github.com/sirupsen/logrus"
  "github.com/sirupsen/logrus/hooks/test"
  "github.com/stretchr/testify/assert"
  "testing"
)

func TestSomething(t*testing.T){
  logger, hook := test.NewNullLogger()
  
  // logger := logrus.New()
  // hook := test.NewLocal(logger)
  
  logger.Error("Helloerror")
  
  assert.Equal(t, 1, len(hook.Entries))
  assert.Equal(t, logrus.ErrorLevel, hook.LastEntry().Level)
  assert.Equal(t, "Helloerror", hook.LastEntry().Message)

  hook.Reset()
  assert.Nil(t, hook.LastEntry())
}
```

## Fatal 处理器

logrus可以定义一个或者多个函数，当记录fatal级别的日志被记录时调用这些函数处理Fatal。注册的函数将在`os.Exit(1)`之前被调用。这种行为有利于优雅退出程序。但是这个机制不能像panic那样使用defer和recover来恢复。

```go
handler := func() {
  // gracefully shutdown something...
}
logrus.RegisterExitHandler(handler)
```

## 线程安全

默认情况下，日志记录器使用mutex互斥锁保护并发写操作。mutex在调用Hook和写日志时被加锁。如果程序是线程安全的，可以调用`logger.SetNoLock()`来禁用互斥锁。

不需要锁的情况包括：

* 未注册Hook，或者Hook是线程安全的。
* 向`logger.Out`写入日志是线程安全的


## 参考链接

[1] [https://github.com/sirupsen/logrus/](https://github.com/sirupsen/logrus/)
