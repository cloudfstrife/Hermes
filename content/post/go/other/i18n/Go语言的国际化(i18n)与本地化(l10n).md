---
title: "Go语言的国际化(i18n)与本地化(l10n)"
date: 2019-01-31T19:21:07+08:00
categories:
- Go
- i18n
tags:
- Go
- i18n
- l10n
keywords:
- Go
- i18n
- l10n
---

Go语言中，大多数消息都是在`fmt`和`template`包中输出，所以，在处理国际化时，也从这两个方向出发。

<!--more-->

`golang.org/x/text`包提供了一些子包，用于处理国际化。本文简单介绍一些常用的场景。

## 消息与Catalogs

消息是要传递给用户的某种形式的内容。每个消息都有一个key,可以有多种语言形式。`message`子包提供了`NewPrinter`函数，此函数返回一个`Printfer`实例，Printer实现了`fmt`风格的API方法。

```go
package main

import (
	"golang.org/x/text/language"
	"golang.org/x/text/message"
)

func main() {
	msg := "happy new year"

	message.SetString(language.SimplifiedChinese, msg, "%d 年快乐\n")
	message.SetString(language.German, msg, "%d Frohes neues jahr\n")
	message.SetString(language.AmericanEnglish, msg, "happy %d\n")

	scp := message.NewPrinter(language.SimplifiedChinese)
	scp.Printf(msg, 2019)
	gp := message.NewPrinter(language.German)
	gp.Printf(msg, 2019)
	aep := message.NewPrinter(language.AmericanEnglish)
	aep.Printf(msg, 2019)
}
```

输出

```text
2,019 年快乐
2.019 Frohes neues jahr
happy 2,019
```

> 注意此处的`2019`不同语言对数字进行了不同的格式化处理，同时使用了不同的语言来输出。

`message.SetString`和`message.NewPrinter`函数都需要一个`Language Tag`参数，`Language Tag`实例可以使用以下方式创建

* 使用预设值

```go
language.SimplifiedChinese
language.German
language.AmericanEnglish
```

* 使用语言字符串创建

```go
language.Make("el")
language.Parse("en-UK")
```

* 组合方式

```go
ja, _ := language.ParseBase("ja")
jp, _ := language.ParseRegion("JP")
jpLngTag, _ := language.Compose(ja, jp)
fmt.Println(jpLngTag)
```

> 如果组织了一个无效的`Language Tag`，方法会返回一个`Und`（表示Undefined Tag）和一个`error`.

为了输出不同语言的消息，需要事先将消息的翻译内容放到`Catalog`中。`message.SetString`和`Printer.Printf`函数输入的`key`是严格判等，如果printf时无法匹配到，将默认使用`key`的内容做为`fmt`方法的参数。

```go
package main

import (
	"golang.org/x/text/language"
	"golang.org/x/text/message"
)

func main() {
	msg := "happy new year"

	message.SetString(language.SimplifiedChinese, msg, "%d 年快乐\n")
	message.SetString(language.German, msg, "%d Frohes neues jahr\n")
	message.SetString(language.AmericanEnglish, msg, "happy %d\n")

	scp := message.NewPrinter(language.SimplifiedChinese)
	scp.Printf(msg, 2019)
	gp := message.NewPrinter(language.German)
	gp.Printf(msg, 2019)
	aep := message.NewPrinter(language.AmericanEnglish)
	aep.Printf("happy new Year", 2019)
}
```

输出

```text
2,019 年快乐
2.019 Frohes neues jahr
happy new Year
```

> 最后一个输出，因为Key不匹配，所以用`happy new Year`做为`fmt.Printf`的参数进行输出。

## catalog.Builder

### 复数的处理

有时程序需要跟据事物的数量进行一些格式化，`golang.org/x/text/feature/plural`导出了一个`SelectF`函数，用于多语言中处理复数。

```go
package main

import (
	"golang.org/x/text/feature/plural"
	"golang.org/x/text/language"
	"golang.org/x/text/message"
)

func main() {
	msg := "eat apple"
	message.Set(
		language.SimplifiedChinese, msg,
		plural.Selectf(
			1, "%d",
			"=1", "我吃了一个苹果\n",
			"=2", "我吃了两个苹果\n",
			"=3", "我吃了三个苹果\n",
			"=4", "我吃了四个苹果\n",
			"=5", "我吃了五个苹果\n",
			"other", "我吃了好多个苹果\n",
		),
	)

	scp := message.NewPrinter(language.SimplifiedChinese)
	scp.Printf(msg, 1)
	scp.Printf(msg, 2)
	scp.Printf(msg, 3)
	scp.Printf(msg, 4)
	scp.Printf(msg, 5)
	scp.Printf(msg, 10)
}
```

输出 

```text
我吃了一个苹果
我吃了两个苹果
我吃了三个苹果
我吃了四个苹果
我吃了五个苹果
我吃了好多个苹果
```

`SelectF`函数的参数有一些变体，比如：`zero` , `one` , `two` , `few` ,`many`，也支持比较操作`>` , `>=` , `<` , `<=`。

### 字符串注入

在处理文本过程中，可以指定占位符变量的方式来处理某些特定的语言特征情况。

```go
package main

import (
	"golang.org/x/text/feature/plural"
	"golang.org/x/text/language"
	"golang.org/x/text/message"
	"golang.org/x/text/message/catalog"
)

func main() {
	msg := "eat apple"
	message.Set(
		language.SimplifiedChinese, msg,
		catalog.Var("number",
			plural.Selectf(1, "%d",
				"=1", "一",
				"=2", "两",
				"=3", "三",
				"=4", "四",
				"=5", "五",
				"other", "好多",
			),
		),
		catalog.String("我吃了 ${number} 个苹果\n"),
	)

	scp := message.NewPrinter(language.SimplifiedChinese)
	scp.Printf(msg, 1)
	scp.Printf(msg, 2)
	scp.Printf(msg, 3)
	scp.Printf(msg, 4)
	scp.Printf(msg, 5)
	scp.Printf(msg, 10)
}
```

输出

```text
我吃了 一 个苹果
我吃了 两 个苹果
我吃了 三 个苹果
我吃了 四 个苹果
我吃了 五 个苹果
我吃了 好多 个苹果
```

`catalog.Var`设置了一个`number`的变量，此变量跟据第一个参数的数值生成变体，在翻译语句中使用`${number}`的形式引用这个变量。

## 货币格式

`golang.org/x/text/currency`包提供了处理货币格式化规则的方法。

```go
package main

import (
	"golang.org/x/text/currency"
	"golang.org/x/text/language"
	"golang.org/x/text/message"
)

func main() {
	msg := "apple price"
	message.SetString(language.SimplifiedChinese, msg, "苹果  : %f / kg\n")
	message.SetString(language.German, msg, "äpfel : %f / kg\n")
	message.SetString(language.AmericanEnglish, msg, "apple : %f / kg\n")

	scp := message.NewPrinter(language.SimplifiedChinese)
	scp.Printf(msg, currency.Symbol(currency.CNY.Amount(100.00)))
	gp := message.NewPrinter(language.German)
	gp.Printf(msg, currency.NarrowSymbol(currency.EUR.Amount(100.00)))
	aep := message.NewPrinter(language.AmericanEnglish)
	aep.Printf(msg, currency.ISO.Kind(currency.Cash)(currency.USD.Amount(100.00)))
}
```

输出 

```text
苹果  : ￥ 100.00 / kg
äpfel : € 100.00 / kg
apple : USD 100.00 / kg
```

以上是常用的一些国际化与本地化的操作，在实际的应用中，消息的翻译往往放在不同的配置文件中方便使用。可以使用配置文件格式解析库解析内容，初始化`catalog`并导出方法供应用调用。

## 常用国家语言代码

| 国家/地区           | 语言代码 |
|:------------------- | :------- |
| 简体中文(中国)      | zh-cn    | 
| 繁体中文(台湾地区)  | zh-tw    |
| 繁体中文(香港)      | zh-hk    | 
| 英语(香港)          | en-hk    |
| 英语(美国)          | en-us    | 
| 英语(英国)          | en-gb    |
| 英语(全球)          | en-ww    | 
| 英语(加拿大)        | en-ca    |
| 英语(澳大利亚)      | en-au    | 
| 英语(爱尔兰)        | en-ie    |
| 英语(芬兰)          | en-fi    | 
| 芬兰语(芬兰)        | fi-fi    |
| 英语(丹麦)          | en-dk    | 
| 丹麦语(丹麦)        | da-dk    |
| 英语(以色列)        | en-il    | 
| 希伯来语(以色列)    | he-il    |
| 英语(南非)          | en-za    | 
| 英语(印度)          | en-in    |
| 英语(挪威)          | en-no    | 
| 英语(新加坡)        | en-sg    |
| 英语(新西兰)        | en-nz    | 
| 英语(印度尼西亚)    | en-id    |
| 英语(菲律宾)        | en-ph    | 
| 英语(泰国)          | en-th    |
| 英语(马来西亚)      | en-my    | 
| 英语(阿拉伯)        | en-xa    |
| 韩文(韩国)          | ko-kr    | 
| 日语(日本)          | ja-jp    |
| 荷兰语(荷兰)        | nl-nl    | 
| 荷兰语(比利时)      | nl-be    |
| 葡萄牙语(葡萄牙)    | pt-pt    | 
| 葡萄牙语(巴西)      | pt-br    |
| 法语(法国)          | fr-fr    | 
| 法语(卢森堡)        | fr-lu    |
| 法语(瑞士)          | fr-ch    | 
| 法语(比利时)        | fr-be    |
| 法语(加拿大)        | fr-ca    | 
| 西班牙语(拉丁美洲)  | es-la    |
| 西班牙语(西班牙)    | es-es    | 
| 西班牙语(阿根廷)    | es-ar    |
| 西班牙语(美国)      | es-us    | 
| 西班牙语(墨西哥)    | es-mx    |
| 西班牙语(哥伦比亚)  | es-co    | 
| 西班牙语(波多黎各)  | es-pr    |
| 德语(德国)          | de-de    | 
| 德语(奥地利)        | de-at    |
| 德语(瑞士)          | de-ch    | 
| 俄语(俄罗斯)        | ru-ru    |
| 意大利语(意大利)    | it-it    | 
| 希腊语(希腊)        | el-gr    |
| 挪威语(挪威)        | no-no    | 
| 匈牙利语(匈牙利)    | hu-hu    |
| 土耳其语(土耳其)    | tr-tr    | 
| 捷克语(捷克共和国)  | cs-cz    |
| 斯洛文尼亚语        | sl-sl    | 
| 波兰语(波兰)        | pl-pl    |
| 瑞典语(瑞典)        | sv-se    | 
| 西班牙语 (智利)     | es-cl    |


## 参考链接

[A Step-by-Step Guide to Go Internationalization (i18n) & Localization (l10n)](https://phraseapp.com/blog/posts/internationalization-i18n-go/)
