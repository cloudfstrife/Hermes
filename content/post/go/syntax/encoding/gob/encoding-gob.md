---
title: "Go encoding/gob"
date: 2019-08-12T12:20:28+08:00
categories:
- Go
tags:
- Go
- encoding
- gob
keywords:
- Go
- encoding
- gob
---

gob是Go语言自带的一个数据序列化的编码/解码工具。类似于 Java 的 `Serialization` 。  
gob和json,xml之类序列化实现一样，使用`Encoder`对数据进行编码，使用`Decoder`解码。

<!--more-->

## 类型与值

Gob 文件或流是完全自描述的：里面包含的所有类型都有一个对应的描述。

整数以两种形式传输：有符号的任意精度整数，无符号的任意精度整数。浮点数始终使用IEEE-754 64位精度发送，目标变量必须能够表示足够的精度，否则解码操作将失败。  
`gob`支持`struct`，`array`和`slice`。`struct`仅对导出的字段进行编码和解码。字符串和字节数组支持特殊，高效的编解码。切片被解码时，如果现有切片具有容量，则切片将被扩展到适当位置;如果没有，则分配新数组。

源值和目标值类型不需要完全对应：对于struct，源值中存在而目标值中缺少的字段将被忽略（以名称为标识）。目标值中存在，但传输类型或者变量中没有的字段，将被忽略。如果两者都有的字段，类型必须是可兼容的。  
`Encoder`和`Decoder`会进行必要的间接操作和解引用，以完成`gobs`与`Go`类型值之间的转换。

### 转换与解引用示例

**源类型**

```go
struct { A, B int }
```

**可编码或者可解码的类型**

```go
struct { A, B int }	// the same
*struct { A, B int }	// extra indirection of the struct
struct { *A, **B int }	// extra indirection of the fields
struct { A, B int64 }	// different concrete value type; see below
struct { B, A int }	// ordering doesn't matter; matching is by name
struct { A, B, C int }	// extra field (C) ignored
struct { B int }	// missing field (A) ignored; data will be dropped
struct { B, C int }	// missing field (A) ignored; extra field (C) ignored.
```

**尝试接收这些类型将返回错误**

```go
struct { A int; B uint }	// change of signedness for B
struct { A int; B float }	// change of type for B
struct { }			// no field names in common
struct { C, D int }		// no field names in common
```

## 主要方法

### 注册

```go
func Register(value interface{})
func RegisterName(name string, value interface{})
```

### 编码

```go
func NewDecoder(r io.Reader) *Decoder
func (dec *Decoder) Decode(e interface{}) error
func (dec *Decoder) DecodeValue(v reflect.Value) error
```

### 解码

```go
func NewEncoder(w io.Writer) *Encoder
func (enc *Encoder) Encode(e interface{}) error
func (enc *Encoder) EncodeValue(value reflect.Value) error
```

## 基础示例

```go
package main

import (
	"bytes"
	"encoding/gob"

	log "github.com/sirupsen/logrus"
)

func init() {
	log.SetFormatter(&log.TextFormatter{
		FullTimestamp: true,
	})
}

type S struct {
	X, Y, Z int
	Name    string
}

type R struct {
	X, Y *int32
	Name string
}

func main() {

	var err error

	buffer := new(bytes.Buffer)
	encoder := gob.NewEncoder(buffer)

	err = encoder.Encode(S{1, 2, 3, "Sending"})
	if err != nil {
		log.Error("gob encoding error : ", err)
	}

	decoder := gob.NewDecoder(buffer)
	r := R{}
	err = decoder.Decode(&r)
	if err != nil {
		log.Error("gob decoding error : ", err)
	}
	log.Infof("receiving data : %d, %d , %s", *(r.X), *(r.Y), r.Name)
}
```

## 切片示例

```go
package main

import (
	"bytes"
	"encoding/gob"

	log "github.com/sirupsen/logrus"
)

func init() {
	log.SetFormatter(&log.TextFormatter{
		FullTimestamp: true,
	})
}

func main() {
	slist := make([]string, 0, 10)
	slist = append(slist, "1", "2", "3")
	log.Infof("SEND - LEN : %d CAP : %d VAL : %#v", len(slist), cap(slist), slist)

	var err error

	buffer := new(bytes.Buffer)
	encoder := gob.NewEncoder(buffer)
	decoder := gob.NewDecoder(buffer)

	err = encoder.Encode(slist)
	if err != nil {
		log.Error("gob encoding error : ", err)
	}

	rlist := make([]string, 0)
	err = decoder.Decode(&rlist)
	if err != nil {
		log.Error("gob decoding error : ", err)
	}
	log.Infof("SEND - LEN : %d CAP : %d VAL : %#v", len(rlist), cap(rlist), rlist)
}

```


## 自定义编码解码器

```go
package main

import (
    "bytes"
    "encoding/gob"
    "fmt"
    "log"
)

// The Vector type has unexported fields, which the package cannot access.
// We therefore write a BinaryMarshal/BinaryUnmarshal method pair to allow us
// to send and receive the type with the gob package. These interfaces are
// defined in the "encoding" package.
// We could equivalently use the locally defined GobEncode/GobDecoder
// interfaces.
type Vector struct {
    x, y, z int
}

func (v Vector) MarshalBinary() ([]byte, error) {
    // A simple encoding: plain text.
    var b bytes.Buffer
    fmt.Fprintln(&b, v.x, v.y, v.z)
    return b.Bytes(), nil
}

// UnmarshalBinary modifies the receiver so it must take a pointer receiver.
func (v *Vector) UnmarshalBinary(data []byte) error {
    // A simple encoding: plain text.
    b := bytes.NewBuffer(data)
    _, err := fmt.Fscanln(b, &v.x, &v.y, &v.z)
    return err
}

// This example transmits a value that implements the custom encoding and decoding methods.
func main() {
    var network bytes.Buffer // Stand-in for the network.

    // Create an encoder and send a value.
    enc := gob.NewEncoder(&network)
    err := enc.Encode(Vector{3, 4, 5})
    if err != nil {
        log.Fatal("encode:", err)
    }

    // Create a decoder and receive a value.
    dec := gob.NewDecoder(&network)
    var v Vector
    err = dec.Decode(&v)
    if err != nil {
        log.Fatal("decode:", err)
    }
    fmt.Println(v)

}
```

> 上面的示例是godoc的自定义编码解码器示例，Vector实现了`encoding.BinaryMarshaler`和`encoding.BinaryUnmarshaler`接口。  
> 以上代码也可以实现`GobDecoder`和`GobEncoder`接口  
> 当都实现时，优先调用`GobDecoder`和`GobEncoder`接口的方法  

**encoding.BinaryMarshaler**

```go
type BinaryMarshaler interface {
    MarshalBinary() (data []byte, err error)
}
```

**encoding.BinaryUnmarshaler**

```go
type BinaryUnmarshaler interface {
    UnmarshalBinary(data []byte) error
}
```


**GobEncoder**

```go
type GobEncoder interface {
    // GobEncode returns a byte slice representing the encoding of the
    // receiver for transmission to a GobDecoder, usually of the same
    // concrete type.
    GobEncode() ([]byte, error)
}
```

**GobDecoder**

```go
type GobDecoder interface {
    // GobDecode overwrites the receiver, which must be a pointer,
    // with the value represented by the byte slice, which was written
    // by GobEncode, usually for the same concrete type.
    GobDecode([]byte) error
}
```

## 接口类型序列化

以下示例展示如何序列化接口值，与常规struct类型的区别在于，使用`func Register(value interface{})`函数注册接口实现类型。

```go
package main

import (
    "bytes"
    "encoding/gob"
    "fmt"
    "log"
    "math"
)

type Point struct {
    X, Y int
}

func (p Point) Hypotenuse() float64 {
    return math.Hypot(float64(p.X), float64(p.Y))
}

type Pythagoras interface {
    Hypotenuse() float64
}

// This example shows how to encode an interface value. The key
// distinction from regular types is to register the concrete type that
// implements the interface.
func main() {
    var network bytes.Buffer // Stand-in for the network.

    // We must register the concrete type for the encoder and decoder (which would
    // normally be on a separate machine from the encoder). On each end, this tells the
    // engine which concrete type is being sent that implements the interface.
    gob.Register(Point{})

    // Create an encoder and send some values.
    enc := gob.NewEncoder(&network)
    for i := 1; i <= 3; i++ {
        interfaceEncode(enc, Point{3 * i, 4 * i})
    }

    // Create a decoder and receive some values.
    dec := gob.NewDecoder(&network)
    for i := 1; i <= 3; i++ {
        result := interfaceDecode(dec)
        fmt.Println(result.Hypotenuse())
    }

}

// interfaceEncode encodes the interface value into the encoder.
func interfaceEncode(enc *gob.Encoder, p Pythagoras) {
    // The encode will fail unless the concrete type has been
    // registered. We registered it in the calling function.

    // Pass pointer to interface so Encode sees (and hence sends) a value of
    // interface type. If we passed p directly it would see the concrete type instead.
    // See the blog post, "The Laws of Reflection" for background.
    err := enc.Encode(&p)
    if err != nil {
        log.Fatal("encode:", err)
    }
}

// interfaceDecode decodes the next interface value from the stream and returns it.
func interfaceDecode(dec *gob.Decoder) Pythagoras {
    // The decode will fail unless the concrete type on the wire has been
    // registered. We registered it in the calling function.
    var p Pythagoras
    err := dec.Decode(&p)
    if err != nil {
        log.Fatal("decode:", err)
    }
    return p
}
```
