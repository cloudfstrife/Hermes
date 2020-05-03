---
title: "TOML"
date: 2019-02-14T18:54:58+08:00
categories:
- TOML
tags:
- TOML
keywords:
- TOML
---

本文是对[https://github.com/toml-lang/toml](https://github.com/toml-lang/toml)的翻译，增加了一些个人理解，还有一些实验中遇到的问题处理。

<!--more-->

![TOML](https://raw.githubusercontent.com/toml-lang/toml/master/logos/toml-200.png)

`TOML`是Github联合创始人`Tom Preston-Werner`(记住这个名字，他很牛X，[他的github地址](https://github.com/mojombo))2013年创建的语言。`TOML`是`Tom’s Obvious, Minimal Language`的缩写, 对应中文为`Tom的浅显、最小化的语言`(这是什么鬼？)。

简单的说，这是一种类似于`INI`、`XML`、`JSON`、`YAML`的数据格式。

`TOML`当前的版本是`v0.5.0`，是一个比较稳定的版本，`v1.0.0`版本会向后尽可能的兼容这个版本。所以，建议所有实现尽量兼容`v0.5.0`，这样在`v1.0.0`发布的时候，会少很多麻烦。

## 设计目标

`TOML`的设计目标是作为一种语意浅显的（Obvious），易读的，最小化（Minimal）的配置文件格式。`TOML`被设计成可以无歧义地被映射为哈希表，可以被多种语言解析。

## 示例

```toml
# This is a TOML document.

title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
dob = 1979-05-27T07:32:00-08:00 # First class dates

[database]
server = "192.168.1.1"
ports = [ 8001, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]

  # Indentation (tabs and/or spaces) is allowed but not required
  [servers.alpha]
  ip = "10.0.0.1"
  dc = "eqdc10"

  [servers.beta]
  ip = "10.0.0.2"
  dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ]

# Line breaks are OK when inside arrays
hosts = [
  "alpha",
  "omega"
]
```

## 规范

* `TOML`是大小写敏感的
* `TOML`文件必须是`UTF-8`编码的
* 空白符可以是空格(0x20)和tab(0x09)
* 换行符可以是LF(0x0A)和CRLF(0x0D0A)

## 注释

`#`之后的内容被标记为注释。

```
# This is a full-line comment
key = "value" # This is a comment at the end of a line
```

## 键值对

`TOML`文档最基本的构成是键/值对。键名在等号的左边，值在右边，键名和键值周围的空格会被忽略。键、等号和值必须在同一行(不过有些值可以跨多行)

键值是必须的，可以是以下类型：

* 字符串
* 整数
* 浮点数
* 布尔值
* 日期时刻
* 数组
* 行内表

### 键名

键名可以是裸键，也可以被引号包裹，被`.`号分隔。

### 裸键

裸键只能包含ASCII字母，ASCII数字，下划线和短横线(`A-Za-z0-9_-`)，可以是纯数学，解析时，这样的键被当做纯数学字符串来处理。

```toml
key = "value"
bare_key = "value"
bare-key = "value"
1234 = "value"
```

### 引号键

引号键（Quoted keys）遵循基本字符串或字面量字符串相同的规则，可以定义更广泛的键名。但是，除非必须使用这样的定义方式才会用，一般情况下裸键就够了。

```toml
"127.0.0.1" = "value"
"character encoding" = "value"
"ʎǝʞ" = "value"
'key2' = "value"
'quoted "value"' = "value"
```

裸键不可以为空，但是空的引号键是可以有(不建议这样用)

```toml
= "no key name"  # INVALID
"" = "blank"     # VALID but discouraged
'' = 'blank'     # VALID but discouraged
```

### 点号键

> 这种方式`v0.5.0`貌似不支持用于`key=value`的形式式，而是只用于表名定义。官方文档没有更新，这里直接翻译了官方的文档。
>
> 如果在`.toml`文件中使用例如`physical.shape = "round"`这样的定义，Golang API会报如下错误：
> 

```toml
Near line 0 (last key parsed ''): bare keys cannot contain '.'
exit status 1
```
 
> 可以改为如下的定义
 
```toml
[physical]
color = "b"
shape = "c"
```

点号键(Dotted keys)是一通过点相连的裸键或引号键。一般用于相似属性，比如数据库用驱动程序名，连接地址，用户名，密码可以使用相同的以点号分隔的字符串做为键名。

```toml
name = "Orange"
physical.color = "orange"
physical.shape = "round"
site."google.com" = true
```

上面的示例等价于`JSON`

```json
{
  "name": "Orange",
  "physical": {
    "color": "orange",
    "shape": "round"
  },
  "site": {
    "google.com": true
  }
}
```

点分隔符周围的空白会被忽略，但是不建议有任何空格。

键名是不可重复的，同时，只要键名没有被**直接定**，就可以使用。

```toml
a.b.c = 1
a.d = 2

# THIS IS INVALID
a.b = 1
a.b.c = 2
```

## 字符串

`TOML`有四种方式来表示字符串：**基本字符串**、**多行基本字符串**、**字面量**和**多行字面量**。所有字符串都**只能**包含有效的`UTF-8`字符 (U+0000 to U+001F, U+007F)。

### 基本字符串(Basic strings)

基本字符串由引号包裹。任何`Unicode`字符都可以使用，除了那些必须转义的：引号，反斜杠，以及控制字符。

```toml
str = "I'm a string. \"You can quote me\". Name\tJos\u00E9\nLocation\tSF."
```

> 常用的转义字符

```text
\b         - backspace       (U+0008)
\t         - tab             (U+0009)
\n         - linefeed        (U+000A)
\f         - form feed       (U+000C)
\r         - carriage return (U+000D)
\"         - quote           (U+0022)
\\         - backslash       (U+005C)
\uXXXX     - unicode         (U+XXXX)
\UXXXXXXXX - unicode         (U+XXXXXXXX)
```

任何`Unicode`字符都可以用`\uXXXX`或`\UXXXXXXXX`的形式来转义。转义码必须是有效的`Unicode`标量值。

所有上面未列出的其它转义序列都是保留的，如果使用，`TOML`应当生成一个错误。

### 多行基本字符串(Multi-line basic strings)

多行基本字符串由三个引号包裹，允许换行，开头引号后的那个换行会被去除。其它空白和换行符会被保留。

```toml
str1 = """
Roses are red
Violets are blue"""
```

`TOML`解析器可以灵活的将多选基本字符串解析成所在平台的换行字符。

```toml
# On a Unix system, the above multi-line string will most likely be the same as:
str2 = "Roses are red\nViolets are blue"

# On a Windows system, it will most likely be equivalent to:
str3 = "Roses are red\r\nViolets are blue"
```

如果某行内容很长，又想写成多行方便阅读，可以在行尾使用`\`。当一行的最后一个非空白字符是`\`时，`TOML`会把它以及它后面的所有空白（包括换行）移除，直到下一个非空白字符或结束引号。

```toml
# The following strings are byte-for-byte equivalent:
str1 = "The quick brown fox jumps over the lazy dog."

str2 = """
The quick brown \


  fox jumps over \
    the lazy dog."""

str3 = """\
       The quick brown \
       fox jumps over \
       the lazy dog.\
       """
```

任何`Unicode`字符都可以使用，除了必须被转义的：反斜杠和控制字符（U+0000 至 U+001F，U+007F）。

### 字面量字符串(Literal strings)

字面量字符串由单引号包裹。类似于基本字符串，只能表现为单行，由于没有转义，无法在由单引号包裹的字面量字符串中写入单引号

```toml
# What you see is what you get.
winpath  = 'C:\Users\nodejs\templates'
winpath2 = '\\ServerX\admin$\system32\'
quoted   = 'Tom "Dubs" Preston-Werner'
regex    = '<\i\c*\s*>'
```

### 多行字面量字符串(Multi-line literal strings)

多行字面量字符串由三个单引号来包裹，允许换行。类似于字面量字符串，无论任何转义都不可用。允许换行，开头引号后的那个换行会被去除，所有其它内容会原样对待。

```toml
regex2 = '''I [dw]on't need \d{2} apples'''
lines  = '''
The first newline is
trimmed in raw strings.
   All other whitespace
   is preserved.
'''
```

## 整数

整数是纯数字。正数可以有加号前缀，负数的前缀是减号。可接受范围为**64**位(记作长整型) [−9,223,372,036,854,775,808 至 9,223,372,036,854,775,807]

```toml
int1 = +99
int2 = 42
int3 = 0
int4 = -17
```

可以在数字之间用下划线来增强可读性。每个下划线两侧必须至少有一个数字。

```toml
int5 = 1_000
int6 = 5_349_221
int7 = 1_2_3_4_5     # VALID but discouraged
```

数字前面不可以增加`0`，`-0`和`+0`是有效的，同于无前缀的`0`。

非负整数也可以用十六进制、八进制或二进制来表示。在这些格式中，数字前面不可以增加0。十六进制值大小写不敏感。数字间可以加下划线(但不能存在于前缀和值之间)

```toml
# hexadecimal with prefix `0x`
hex1 = 0xDEADBEEF
hex2 = 0xdeadbeef
hex3 = 0xdead_beef

# octal with prefix `0o`
oct1 = 0o01234567
oct2 = 0o755 # useful for Unix file permissions

# binary with prefix `0b`
bin1 = 0b11010110
```
## 浮点数 

浮点数应当实现`IEEE 754`64位双精度值。浮点数由整数部分（遵从与十进制整数值相同的规则）后跟小数部分(小数部分是小数点后跟一个或多个数字)和/或指数部分(指数部分是一个 E(大小写都可以)后跟一个整数)组成。如果小数部分和指数部分兼有，那小数部分必须在指数部分前面。

```toml
# fractional
flt1 = +1.0
flt2 = 3.1415
flt3 = -0.01

# exponent
flt4 = 5e+22
flt5 = 1e6
flt6 = -2E-2

# both
flt7 = 6.626e-34
```

与整数相似，可以使用下划线来增强可读性。

```toml
flt8 = 224_617.445_991_228
```

特殊浮点数表示

```toml
# infinity 无穷大
sf1 = inf  # positive infinity 正无穷大
sf2 = +inf # positive infinity 正无穷大
sf3 = -inf # negative infinity 负无穷小

# not a number 非数
sf4 = nan  # actual sNaN/qNaN encoding is implementation specific
sf5 = +nan # same as `nan`
sf6 = -nan # valid, actual encoding is implementation specific
```

## 布尔值

布尔值只有`true`和`false`，全部小写。

```toml
bool1 = true
bool2 = false
```
## 日期与时间

## 偏移日期与时间(Offset Date-Time)

特定的时间可以使用指定了偏移量的`RFC 3339`格式的表示。

```toml
odt1 = 1979-05-27T07:32:00Z
odt2 = 1979-05-27T00:32:00-07:00
odt3 = 1979-05-27T00:32:00.999999-07:00
```

为了便于阅读，可以将日期和时间之间的`T`分隔符替换为空格(RFC 3339第5.6节所允许这样的格式)

```toml
odt4 = 1979-05-27 07:32:00Z
```

小数秒的精度取决于实现，但至少应当精确到毫秒。如果值的精度超出了实现所支持的精度，那多余的部分必须被舍弃，而**不能四舍五入**。

### 当地日期与时间

如果省略`RFC 3339`日期时间格式中的时区偏移量，就表示该日期时间的使用不涉及时区偏移(没有时区当然无法转换成当地时间)。

```toml
ldt1 = 1979-05-27T07:32:00
ldt2 = 1979-05-27T00:32:00.999999
```

### 当地日期

如果只包含`RFC 3339`日期时间格式的日期部分，那么表示一整天，与偏移量或时区没有任何关系。

```toml
ld1 = 1979-05-27
```

### 当地时间

如果只包含`RFC 3339`日期时间格式的时间部分，那么表示一天中的时间，与特定的一天没有任何关系，同时也没有任何偏移量或时区。

```toml
lt1 = 07:32:00
lt2 = 00:32:00.999999
```

## 数组

数组是一组被**方括号**包裹的相同类型的值。元素由逗号分隔，元素之间的空白会被忽略，元素的数据类型必须一致(不同写法的字符串被认为是相同的类型，不同元素类型的数组也被认为是同样的数组类型)。

```toml
arr1 = [ 1, 2, 3 ]
arr2 = [ "red", "yellow", "green" ]
arr3 = [ [ 1, 2 ], [3, 4, 5] ]
arr4 = [ "all", 'strings', """are the same""", '''type''']
arr5 = [ [ 1, 2 ], ["a", "b", "c"] ]

arr6 = [ 1, 2.0 ] # INVALID
```

数组也可以跨多行，最后一个值后面可以有逗号(称为尾逗号)，值和结束括号前可以存在任意数量的换行和注释。

```toml
arr7 = [
  1, 2, 3
]

arr8 = [
  1,
  2, # this is ok
]
```

## 表

表(又称为哈希表或者字典)是一组键值对的集合。

表名由**方括号**包裹，单独一行出现，表名的规则与键名相同。

```toml
[table]
```

在表名下方至下一个表或文件结束之间的键值对，都是这个表的键值对。表不保证这些键值对的顺序。

```toml
[table-1]
key1 = "some string"
key2 = 123

[table-2]
key1 = "another string"
key2 = 456

[dog."tater.man"]
type.name = "pug"
```

上面最后一个示例等价于`JSON`的如下结构：

```json
{ "dog": { "tater.man": { "type": { "name": "pug" } } } }
```

`TOML`可以定义空表，但是不允许重复定义表名，同时，只要一个表名还没有被直接定义过，就可以对它和它下属的键名赋值。

```toml
# DO NOT DO THIS

[a]
b = 1

[a]
c = 2
```

```
# DO NOT DO THIS EITHER

[a]
b = 1

[a.b]
c = 2
```

## 行内表

行内表使用一种更紧凑的语法来表示表格。

行内表由**花括号**包裹，在花括号中，可以出现零个或更多逗号分隔的键值对。键值对采用与标准表中的键值对相同的形式。

行内表不允许换行(多行字符串除外)，即使是这样，也强烈不建议让一个行内表跨多行(不然干嘛叫行内表？)。

```toml
name = { first = "Tom", last = "Preston-Werner" }
point = { x = 1, y = 2 }
animal = { type.name = "pug" }
```

上面的行内表等同于下面的标准表定义

```toml
[name]
first = "Tom"
last = "Preston-Werner"

[point]
x = 1
y = 2

[animal]
type.name = "pug"
```

## 表数组

表数组使用**双方括号**来表示，具有相同方括号名的表将会成为该数组内的成员。元素的顺序就是元素在文件中出现顺序。没有键值对的双方括号表将被视为一个空表。

```toml
[[products]]
name = "Hammer"
sku = 738594937

[[products]]

[[products]]
name = "Nail"
sku = 284758393
color = "gray"
```

等价的`JSON`如下：

```json
{
  "products": [
    { "name": "Hammer", "sku": 738594937 },
    { },
    { "name": "Nail", "sku": 284758393, "color": "gray" }
  ]
}
```

表数组可以嵌套，在子表是使用相同的语法即可。

```toml
[[fruit]]
  name = "apple"

  [fruit.physical]
    color = "red"
    shape = "round"

  [[fruit.variety]]
    name = "red delicious"

  [[fruit.variety]]
    name = "granny smith"

[[fruit]]
  name = "banana"

  [[fruit.variety]]
    name = "plantain"
```

等价的`JSON`如下：

```json
{
  "fruit": [
    {
      "name": "apple",
      "physical": {
        "color": "red",
        "shape": "round"
      },
      "variety": [
        { "name": "red delicious" },
        { "name": "granny smith" }
      ]
    },
    {
      "name": "banana",
      "variety": [
        { "name": "plantain" }
      ]
    }
  ]
}
```

不要试图向一个静态定义的数组追加内容，那一定会报错的。即使数组为空或者类型兼容。

```toml
# INVALID TOML DOC
fruit = []

[[fruit]] # Not allowed
```

可以适当使用行内表

```toml
points = [ { x = 1, y = 2, z = 3 },
           { x = 7, y = 8, z = 9 },
           { x = 2, y = 4, z = 8 } ]
```

## 文件扩展名 

`TOML`文件使用 .toml 扩展名。在互联网上传输 TOML 文件时，MIME类型为`application/toml`。



## 参考链接

[https://github.com/toml-lang/toml/](https://github.com/toml-lang/toml/)  
[https://github.com/toml-lang/toml/blob/master/versions/cn/toml-v0.5.0.md](https://github.com/toml-lang/toml/blob/master/versions/cn/toml-v0.5.0.md)
