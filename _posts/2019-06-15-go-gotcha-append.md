---
layout: post
title: go append 使用陷阱
tags: go gotcha
---

### 问题
请大家思考下如下代码的输出结果,

```
package main

import "fmt"

func main() {
	a := []byte("aa")
	b := append(a, 'b')
	c := append(a, 'c')
	fmt.Println(string(a), len(a), cap(a), &a, &a[0])
	fmt.Println(string(b), len(b), cap(b), &b, &b[0])
	fmt.Println(string(c), len(c), cap(c), &c, &c[0])
}
```
实际输出结果为:

```
aa 2 8 &[97 97] 0xc000016090
aac 3 8 &[97 97 99] 0xc000016090
aac 3 8 &[97 97 99] 0xc000016090
```
### 原因

b,c 通过a append后， a,b,c三个slice共用底层的数组，也就是说,
a,b,c三个slice的data字段指向同一个底层数组,
所以对任意一个slice的修改， 都会影响其他的slice的值。

go中切片对应一个数据结构，如下:

```
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```




