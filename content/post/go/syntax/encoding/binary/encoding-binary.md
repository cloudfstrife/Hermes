---
title: "Go encoding/binary"
date: 2019-07-18T18:32:25+08:00
categories:
- Go
tags:
- Go
- encoding
- binary
keywords:
- Go
- encoding
- binary
---

`encoding/binary`包实现了数字与`[]byte`之前的简易转换，及其变体的编码与解码。  
`encoding/binary`包更倾向于易用，而不是高性能，如果需要一个高性能的序列化或者需要处理大量数据，请优先考虑`encoding/gob`或者`protocol buffers`。

<!--more-->

在说数字转换成`[]byte`之前，要先明白一个概念：**字节序**

## 字节序

计算机硬件有两种储存数据的方式：**大端字节序(big endian)**和**小端字节序(little endian)**。

* 大端字节序：高位字节在前，低位字节在后
* 小端字节序：低位字节在前，高位字节在后

**举例**

0x1234567  
大端字段序为

```text
| 0X100 | 0X101 | 0X102 | 0X103 |						//内存位置
| 01    | 23    | 45    | 67    |						//存储值
```

小端字节序为

```text
| 0X100 | 0X101 | 0X102 | 0X103 |						//内存位置
| 67    | 45    | 23    | 01    |						//存储值
```

是日常使用中，只需要在读取数据时确定数据的字节序即可，其它时间不需要考虑。处理器读取数据时，必须知道数据的字节序，才能将其转成正确的值。

Go语言中的`binary.BigEndian`和`binary.LittleEndian`定义了编解码时使用的字节序。  
日常开发中，数据生产者与数据消费者应该使用相同的字节序来编解码数据，防止因为字节序不同造成数据错误。



## 编码解码

**示例**

```go
package main

import (
	"encoding/binary"

	log "github.com/sirupsen/logrus"
)

func main() {
	v := make([]byte, 14)
	binary.BigEndian.PutUint16(v[0:], 1)
	binary.BigEndian.PutUint32(v[2:], 2)
	binary.BigEndian.PutUint64(v[6:], 3)
	log.Printf("%08b", v)

	u16 := binary.BigEndian.Uint16(v[0:2])
	u32 := binary.BigEndian.Uint32(v[2:6])
	u64 := binary.BigEndian.Uint64(v[6:])
	log.Printf("%d", u16)
	log.Printf("%d", u32)
	log.Printf("%d", u64)
}
```

每次方法调用产生的binary长度是可确定的，所以数据可以通过计算存储到slice的不同的位置。

## 对流进行编码与解码

**示例**

```go
package main

import (
	"bytes"
	"encoding/binary"

	log "github.com/sirupsen/logrus"
)

func main() {
	w := bytes.NewBuffer(make([]byte, 0))
	binary.Write(w, binary.BigEndian, uint16(1))
	binary.Write(w, binary.BigEndian, uint32(2))
	binary.Write(w, binary.BigEndian, uint64(3))
	log.Printf("%08b", w.Bytes())

	var err error
	var u16 uint16
	err = binary.Read(w, binary.BigEndian, &u16)
	if err != nil {
		log.Error(err)
	}
	log.Printf("%d", u16)

	var u32 uint32
	err = binary.Read(w, binary.BigEndian, &u32)
	if err != nil {
		log.Error(err)
	}
	log.Printf("%d", u32)

	var u64 uint64
	err = binary.Read(w, binary.BigEndian, &u64)
	if err != nil {
		log.Error(err)
	}
	log.Printf("%d", u64)
}

```

## struct编码解码

`binary.Write`方法可以编码包含**固定长度值**的数组，切片和结构体。

固定长度值：是指可以明确确定底层存储二进制的字节数的类型的值。

**结构体示例**


```go
package main

import (
	"bytes"
	"encoding/binary"

	log "github.com/sirupsen/logrus"
)

//SensorReport 传感器上报数据包
type SensorReport struct {
	//ID 传感器ID
	ID uint32
	//AirPressure 气压
	AirPressure uint32
	//Temperature 温度
	Temperature uint32
	//Longitude 经度
	Longitude float64
	//Latitude 纬度
	Latitude float64
}

func main() {
	w := bytes.NewBuffer(make([]byte, 0))
	wSReport := &SensorReport{
		ID:          1,
		AirPressure: 1000,
		Temperature: 35,
		Longitude:   1.23,
		Latitude:    4.56,
	}
	binary.Write(w, binary.BigEndian, wSReport)
	log.Printf("%08b", w.Bytes())

	rSReport := new(SensorReport)

	err := binary.Read(w, binary.BigEndian, rSReport)
	if err != nil {
		log.Error(err)
	}
	log.Infof("传感器ID:\t%d", rSReport.ID)
	log.Infof("气压:\t%d", rSReport.AirPressure)
	log.Infof("温度:\t%d", rSReport.Temperature)
	log.Infof("经度:\t%f", rSReport.Longitude)
	log.Infof("纬度:\t%f", rSReport.Latitude)
}

```

**结构体切片**

```go
package main

import (
	"bytes"
	"encoding/binary"

	log "github.com/sirupsen/logrus"
)

//SensorReport 传感器上报数据包
type SensorReport struct {
	//ID 传感器ID
	ID uint32
	//AirPressure 气压
	AirPressure uint32
	//Temperature 温度
	Temperature int32
	//Longitude 经度
	Longitude float64
	//Latitude 纬度
	Latitude float64
}

func main() {
	w := bytes.NewBuffer(make([]byte, 0))
	wSReports := []SensorReport{
		{
			ID:          1,
			AirPressure: 1000,
			Temperature: 35,
			Longitude:   1.23,
			Latitude:    4.56,
		},
		{
			ID:          2,
			AirPressure: 1500,
			Temperature: -5,
			Longitude:   7.89,
			Latitude:    0.12,
		},
		{
			ID:          3,
			AirPressure: 1200,
			Temperature: 20,
			Longitude:   3.45,
			Latitude:    6.78,
		},
	}
	binary.Write(w, binary.BigEndian, wSReports)
	log.Printf("%08b", w.Bytes())

	rSReports := make([]SensorReport, 3)

	err := binary.Read(w, binary.BigEndian, rSReports)
	if err != nil {
		log.Error(err)
	}
	for _, rSReport := range rSReports {
		log.Infof("传感器ID:\t%d", rSReport.ID)
		log.Infof("气压:\t%d", rSReport.AirPressure)
		log.Infof("温度:\t%d", rSReport.Temperature)
		log.Infof("经度:\t%f", rSReport.Longitude)
		log.Infof("纬度:\t%f", rSReport.Latitude)
		log.Info("------------------------------------")
	}
}

```
