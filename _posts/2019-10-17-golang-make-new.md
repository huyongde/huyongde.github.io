---
layout: post
title: golang中 make 和 new 关键字的实现
tags: go
---

#### 下图概括了make 和 new 关键字的实现:
![make_new](/image/golang_make_new.png)

## make vs new

### make

- 1. make 对应的 OP type 是 OMAKE, 在类型检查阶段 typecheck1()函数会把 OMAKE 根据参数类型调整为 OMAKESLICE或 OMAKECHAN或 OMAKEMAP

  cmd/compile/internal/gc/typecheck.go

- 2. walkexpr()函数会通过 mkcall()函数把 OMAKECHAN、OMAKEMAP、OMAKESLICE 分别调用运行时函数 makechan、makemap、makeslice

  cmd/compile/internal/gc/walk.go

- 返回值

	- makeslice 返回 slice 结构体

	  runtime/slice.go

	- makemap 返回 hmap 结构体的指针

	  runtime/hashmap.go

	- makechan 返回hchan 结构体的指针

	  runtime/chan.go

- makeslice

	- 通过 mallocgc分配内存并返回 slice 结构体

- makemap

	- 通过 mallocgc 申请内存创建 hmap 结构体，并为 hmap 结构体的成员buckets 和 overflow申请内存，最后返回 hmap 的指针

- makechan

	- makechan 做了优化，分了三类 case,
	- 1. chan 无 buf 时，只需要分配 hchan 结构体的大小并返回 hchan的指针
	- 2. chan 有 buf并且元素是无指针的基础类型，把 hchan 结构体和  hchan 中buf的空间分配在一个连续的内存空间中，并返回 hchan 的指针
	- 3. chan 有 buf 并且元素含指针，分别分配 hchan 结构体和结构体中 buf 的内存

### new

- new 对应的 OP type 是 ONEW,walkexpr()会根据是否需要逃逸到堆上来分别处理，需要逃逸到堆上的话调用 callnew(),不需要的话直接在栈上分配

  cmd/compile/internal/gc/walk.go

- callnew()中会通过 mkcall1函数调用运行时函数 newobject()

  cmd/compile/internal/gc/walk.go

- newobject()中只是调用了 mallocgc 申请内存并返回内存的指针,mallocgc()函数中涉及到了 go 的内存管理机制，后面详细介绍

  runtime/malloc.go

- 返回值: new 的返回值都是指针


> 如上是对 make 和 new 实现的整理，可参考着去看下golang 源码，加深理解。


### go 的版本信息

```
$ go version
go version go1.13 darwin/amd64
```

