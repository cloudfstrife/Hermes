---
title: "Go语言container/ring"
date: 2021-04-09T09:44:05+08:00
categories:
- go
tags:
- go
- container
- ring
keywords:
- go
- container
- ring
---

Go语言的 `container/ring` 包实现了环形链表的操作。

<!--more-->

## container/ring 的API

```go
type Ring
    func New(n int) *Ring                           # 创建一个ring
    func (r *Ring) Do(f func(interface{}))          # 循环环上的元素，并进行操作
    func (r *Ring) Len() int                        # 环的长度
    func (r *Ring) Link(s *Ring) *Ring              # 连接两个环
    func (r *Ring) Move(n int) *Ring                # 移动当前指针，n为负值时后移，n为正值时前移
    func (r *Ring) Next() *Ring                     # 指针前移
    func (r *Ring) Prev() *Ring                     # 指针后移
    func (r *Ring) Unlink(n int) *Ring              # 从 r.Next（）开始。删除 n% r.Len（） 个元素，返回值为删除的元素
```

## 示例

```go
package main

import (
	"container/ring"
	"fmt"
)

func main() {
	l := 10

	pf := func(p interface{}) {
		fmt.Printf("%2d ", p.(int))
	}

	// 初始化一个环
	r := ring.New(l)
	for i := 1; i <= l; i++ {
		r.Value = i
		r = r.Next()
	}
	r.Do(pf)
	fmt.Println()

	// 创建另一个环
	o := ring.New(l)
	for i := 1; i <= l; i++ {
		o.Value = i + 90
		o = o.Next()
	}

	// 合并环
	r.Link(o)
	r.Do(pf)
	fmt.Println()

	// 删除
	rmvd := r.Unlink(15)
	r.Do(pf)
	fmt.Println()

	rmvd.Do(pf)
	fmt.Println()
}
```