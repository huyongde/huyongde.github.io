---
title: go服务优化技巧
tag: go 
layout: post
---

### 简介 
本文介绍go服务可能会用到的几个优化技巧

### 技巧1: sync.Pool 池化某些对象，实现复用
池化后进行对象复用，可以减少对象重复创建的开销，并且可以减轻gc的压力。

使用示例:
```
package main

import (
	"fmt"
	"sync"
)

var bufpool = sync.Pool{
	New: func() interface{} {
		buf := make([]byte, 0, 512)
		return &buf
	},
}

func main() {
	b1 := *bufpool.Get().(*[]byte)
	b1 = append(b1, []byte("aaaaa")...)
	fmt.Println(b1, len(b1), cap(b1))
	fmt.Printf("%p, %p \n", b1, &b1)
	bufpool.Put(&b1)
	b2 := *bufpool.Get().(*[]byte)
	fmt.Println(b2, len(b2), cap(b2))
	fmt.Printf("%p, %p \n", b2, &b2)
	bufpool.Put(&b2)
}
```

示例程序输出结果为:
```
[97 97 97 97 97] 5 512
0xc4200a6000, 0xc4200a2020
[97 97 97 97 97] 5 512
0xc4200a6000, 0xc4200a20a0

```

b2和b1对应的底层页数组的地址是一致的，并且b2从pool里取出来时，保留了b1的值，这样是不合理的，
b2的预期值应该是个空对象，所以在put之前需要把对象归零。
如上示例，在`bufpoo.Put(&b1)` 之前增加`b1=b1[0:0]` 后输出结果如下:

```
[97 97 97 97 97] 5 512
0xc420096000, 0xc42000a060
[] 0 512
0xc420096000, 0xc42000a0e0
```
符合预期

### 技巧2: 避免用带有指针的结构体对象做大map的key
用带指针的对象做map的key, 在gc时会耗费更多的时间，因为gc需要根据指针去遍历所有的数据。
比如`map[string]int` string 做map的可以，string在go里用如下结构体实现:

```
type StringHeader struct {
    Data uintptr
    Len  int
}
```
详细介绍在[StringHeader](https://golang.org/src/reflect/value.go?s=56526:56578#L1873)
string中是包含指针的，所以相比用无指针的对象做key，gc会更耗时。
#### 示例:
```
package main

import (
    "fmt"
    "runtime"
    "strconv"
    "time"
)

const numElements = 1000000

var foo = map[string]int{}

func case1() {
    for i := 0; i < numElements; i++ {
        foo[strconv.Itoa(i)] = i
    }

}

var foo2 = map[int]int{}

func case2() {
    for i := 0; i < numElements; i++ {
        foo2[i] = i
    }
}
func timeGC() {
    t := time.Now()
    runtime.GC()
    fmt.Println("gc took time:", time.Since(t))
}
func main() {
    case1()
    //case2()
    for {
        timeGC()
        time.Sleep(1 * time.Second)
    }

}

```
注释case2()， 打开case1()时，输出如下:
```
gc took time: 40.927788ms
gc took time: 40.265383ms
gc took time: 40.235497ms
gc took time: 40.562543ms
gc took time: 41.379995ms
gc took time: 40.582498ms
gc took time: 42.926792ms
```

注释case1(), 打开case2()时，输出如下:
```
gc took time: 285.715µs
gc took time: 159.778µs
gc took time: 158.922µs
gc took time: 168.993µs
gc took time: 159.776µs
gc took time: 175.365µs
```

gc耗时差别巨大。

所以在使用大map时，尽量避免使用带指针的结构体对象做key。

### 技巧3:  使用 strings.Builder 来构拼接字符串
Go 1.10 版本, 提供了`strings.Builder` 来更高效的进行字符串的拼接，
Builder 底层实现是向一个byte 的 buffer 中不断写入数据.
Builder 实现在`src/string/builder.go` 中

通过例子对比下性能差异

```
// main.go
package main

import "strings"

var strs = []string{
	"here's",
	"a",
	"some",
	"long",
	"list",
	"of",
	"strings",
	"for",
	"you",
}

func buildStrNaive() string {
	var s string

	for _, v := range strs {
		s += v
	}

	return s
}
func buildStrBuilder(grow bool) string {
	b := strings.Builder{}
	if grow {
		b.Grow(60)
	}
	for _, v := range strs {
		b.WriteString(v)
	}
	return b.String()
}
```
```
// main_test.go
package main

import "testing"

func BenchmarkBuildStr(b *testing.B) {
	b.Run("Naive", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			buildStrNaive()
		}
	})
	b.Run("builder-0", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			buildStrBuilder(false)
		}
	})
	b.Run("builder-1", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			buildStrBuilder(true)
		}
	})
}
```

通过`go test -bench=. -benchmem`进行基准测试，结果如下

```
goos: darwin
goarch: amd64
BenchmarkBuildStr/Naive-4         	 3424706	       374 ns/op	     216 B/op	       8 allocs/op
BenchmarkBuildStr/builder-0-4     	 7817630	       176 ns/op	     120 B/op	       4 allocs/op
BenchmarkBuildStr/builder-1-4     	17136417	        67.4 ns/op	      64 B/op	       1 allocs/op
PASS
```
在通过 `Builder.Grow()` 提前预分配空间的情况下性能提升了4倍, 即使不提前预分配空间也能提升一倍多。


### 技巧4:  使用strconv包替代fmt包
在把整数转为字符串时，`strconv.Itoa`性能会比`fmt.Sprintf`好很多，`fmt.Sprintf` 使用接口interface{}作为参数，存在如下缺点:
* 失去了类型安全
* 变量转为interface{}时会进行内存申请

接下来， 通过基准测试对比两种把整数转为字符串的方法

```
package main

import (
	"fmt"
	"strconv"
)

func strconvFmt(b int) string {
	return strconv.Itoa(b)
}
func fmtFmt(b int) string {
	return fmt.Sprintf("%d", b)
}
func main(){}
```

test文件:

```
func BenchmarkFmt(b *testing.B) {
	big := 10000
	small := 10
	b.Run("strconv_small", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			strconvFmt(small)
		}
	})
	b.Run("strconv_big", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			strconvFmt(big)
		}
	})
	b.Run("fmt.Sprintf_small", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			fmtFmt(small)
		}
	})
	b.Run("fmt.Sprintf_big", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			fmtFmt(big)
		}
	})

}
```
执行`go test -bench=BenchmarkFmt -benchmem`得到如下结果
```
goos: darwin
goarch: amd64
BenchmarkFmt/strconv_small-4         	305850991	         3.92 ns/op	       0 B/op	       0 allocs/op
BenchmarkFmt/strconv_big-4           	30948387	        32.7 ns/op	       5 B/op	       1 allocs/op
BenchmarkFmt/fmt.Sprintf_small-4     	10915570	       105 ns/op	      16 B/op	       2 allocs/op
BenchmarkFmt/fmt.Sprintf_big-4       	10358809	       113 ns/op	      16 B/op	       2 allocs/op
PASS
```

`strconv.Itoa`的性能是`fmt.Sprintf`的3倍多，并且`strconv.Itoa`在处理绝对值小于100的整数时做了优化，
不需要进行alloc操作，性能更高。详情可以看`src/strconv/itoa.go`中的代码。

### 技巧5 []byte 转 string 时，用 unsafe 包

`string(byteSlice)` 把`[]byte` 转为 `string` 对应的是`OARRAYBYTESTR`操作 (`src/cmd/compile/internal/gc/walk.go`,`src/cmd/compile/internal/gc/syntax.go`)，
此操作在编译阶段会映射成运行时函数`slicebytetostring`,  此函数定义在`src/runtime/string.go` 中，
需要进行内存申请，性能会有影响。
下面通过基准测试对比下:

```
func BenchmarkTostr(b *testing.B) {
	bs := []byte("hello go")
	var str string
	b.Run("unsafe", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			str = *(*string)(unsafe.Pointer(&bs))
		}

	})
	b.Run("normal", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			str = string(bs)
		}

	})
	fmt.Println(str)
}
```
基准测试结果:

```
goos: darwin
goarch: amd64
BenchmarkTostr/unsafe-4         	1000000000	         0.789 ns/op	       0 B/op	       0 allocs/op
BenchmarkTostr/normal-4         	64188240	        16.3 ns/op	       8 B/op	       1 allocs/op
hello go
PASS
```
从测试结果看，多了一次内存分配，性能差了20多倍
所以在某些情况下，可以考虑使用 unsafe 包实现`[]byte` 转为`string`


### 说明
go 的版本信息为:
```
$ go version
go version go1.13 darwin/amd64
```
### 参考
* [Simple techniques to optimise Go programs](https://stephen.sh/posts/quick-go-performance-improvements)

