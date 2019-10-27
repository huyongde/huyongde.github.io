---
layout: post
title: golang中 select 关键字的实现
tags: go
---

#### 下图概括了make 和 new 关键字的实现:
![select](/image/golang-select.png)

i## select
在go中，通过select可以实现等待多个channel达到就绪状态，select中的case都要关联到channel相关的读写操作。

select 中每个case都会生成一个对应的scase结构体，结构体定义如下
```
// Select case descriptor.
// Known to compiler.
// Changes here must also be made in src/cmd/internal/gc/select.go's scasetype.
type scase struct {
    c           *hchan         // chan
    elem        unsafe.Pointer // data element
    kind        uint16  
    pc          uintptr // race pc (for race detector / msan)
    releasetime int64
}
```

case的类型有如下几种:

```
// scase.kind values.
// Known to compiler.
// Changes here must also be made in src/cmd/compile/internal/gc/select.go's walkselect.
const (
    caseNil = iota
    caseRecv
    caseSend
    caseDefault
)
```

### 编译阶段
select对应的opType是`OSELECT`, 当`sop==OSELECT`时，会调用`walkselect()`，代码在`src/cmd/compile/internal/gc/walk.go`中， 如下

```
// The result of walkstmt MUST be assigned back to n, e.g.
//  n.Left = walkstmt(n.Left)
func walkstmt(n *Node) *Node {
    if n == nil {
        return n
    }
    ......
     case OSELECT:
        walkselect(n)

    case OSWITCH:
        walkswitch(n)

    case ORANGE:
        n = walkrange(n)
    }

    if n.Op == ONAME {
        Fatalf("walkstmt ended up with name: %+v", n)
    }
    return n
}
```
其中省略了大部分代码。

`walkselect()`定义在`src/cmd/compile/internal/gc/select.go`中，其中会调用`walkselectcases()`,其中会生成一个scase的数组，
并调用运行时的函数`selectgo()`， 定义在`src/runtime/select.go`中

walkselect()源码如下:

```
func walkselect(sel *Node) {
    lno := setlineno(sel)
    if sel.Nbody.Len() != 0 {
        Fatalf("double walkselect")
    }

    init := sel.Ninit.Slice()
    sel.Ninit.Set(nil)

    init = append(init, walkselectcases(&sel.List)...)
    sel.List.Set(nil)

    sel.Nbody.Set(init)
    walkstmtlist(sel.Nbody.Slice())

    lineno = lno
}
```
#### walkselectcases()
`walkselectcases()`中会分如下几种情况处理select：
1. select中不存在case, 直接堵塞
2. select中仅存在一个case
3. select中存在两个case，其中一个是default
4. 其他select情况如: 包含多个case并且有default等

前三种情况不会走到`selectgo()`的逻辑，最后一种多个case的情况会调用运行时函数`selectgo()`

`walkselectcases()` 部分源码如下:

```

func walkselectcases(cases *Nodes) []*Node {
    n := cases.Len()
    sellineno := lineno

    // optimization: zero-case select
    if n == 0 {
        return []*Node{mkcall("block", nil, nil)}
    }

    // optimization: one-case select: single op.
    // TODO(rsc): Reenable optimization once order.go can handle it.
    // golang.org/issue/7672.
    if n == 1 {
        cas := cases.First()
        setlineno(cas)
        l := cas.Ninit.Slice()
        ......
   	}
   	// optimization: two-case select but one is default: single non-blocking op.
    if n == 2 && (cases.First().Left == nil || cases.Second().Left == nil) {
        var cas *Node
        var dflt *Node
        if cases.First().Left == nil {
            cas = cases.Second()
            dflt = cases.First()
        } else {
            dflt = cases.Second()
            cas = cases.First()
        }
        .......
    }
    var init []*Node

    // generate sel-struct
    lineno = sellineno
    selv := temp(types.NewArray(scasetype(), int64(n)))
    r := nod(OAS, selv, nil)
    r = typecheck(r, ctxStmt)
    init = append(init, r)

    order := temp(types.NewArray(types.Types[TUINT16], 2*int64(n)))
    r = nod(OAS, order, nil)
    r = typecheck(r, ctxStmt)
    init = append(init, r)

    // register cases
    for i, cas := range cases.Slice() {
        setlineno(cas)

        init = append(init, cas.Ninit.Slice()...)
        cas.Ninit.Set(nil)

        // Keep in sync with runtime/select.go.
        const (
            caseNil = iota
            caseRecv
            caseSend
            caseDefault
        )
        .......
    }


    // run the select
    lineno = sellineno
    chosen := temp(types.Types[TINT])
    recvOK := temp(types.Types[TBOOL])
    r = nod(OAS2, nil, nil)
    r.List.Set2(chosen, recvOK)
    fn := syslook("selectgo")
    r.Rlist.Set1(mkcall1(fn, fn.Type.Results(), nil, bytePtrToIndex(selv, 0), bytePtrToIndex(order, 0), nodintconst(int64(n))))
    r = typecheck(r, ctxStmt)
    init = append(init, r)

    // selv and order are no longer alive after selectgo.
    init = append(init, nod(OVARKILL, selv, nil))
    init = append(init, nod(OVARKILL, order, nil))

    // dispatch cases
    for i, cas := range cases.Slice() {
        setlineno(cas)

        cond := nod(OEQ, chosen, nodintconst(int64(i)))
        cond = typecheck(cond, ctxExpr)
        cond = defaultlit(cond, nil)

        r = nod(OIF, cond, nil)

        if n := cas.Left; n != nil && n.Op == OSELRECV2 {
            x := nod(OAS, n.List.First(), recvOK)
            x = typecheck(x, ctxStmt)
            r.Nbody.Append(x)
        }

        r.Nbody.AppendNodes(&cas.Nbody)
        r.Nbody.Append(nod(OBREAK, nil, nil))
        init = append(init, r)
    }

    return init
}

```
#### 运行时函数selectgo()
`selectgo()`的主要逻辑:
1. 随机生成轮询顺序`poolorder`,
2. 按照 channel 地址生成锁定顺序`lockorder`
3. 根据 `poolorder` 遍历所有的 `case` 看是否有可以立即处理的 `channel` 读写操作，有的话直接返回
4. 创建 `sudog` 结构体，并加入到 `chan`的 `sendq` 或者 `recvq`,并通过 `gopark` 触发调度器进行调度，当前协程进入 `waiting`状态
5. 堵塞并等待被唤醒
6. 当前协程被唤醒时，再次按照 `lockorder` 遍历所有的 `case`,从中查找要处理的 `sudog` 结构，并释放其他 `sudog`，并返回要处理的`sudog` 对应的 `scase` 的索引

通过源码中的注释，可以很清晰的看到如上的主要逻辑。

详细代码在`src/runtime/select.go`中, 主要部分代码如下：
```
// selectgo implements the select statement.
//
// cas0 points to an array of type [ncases]scase, and order0 points to
// an array of type [2*ncases]uint16. Both reside on the goroutine's
// stack (regardless of any escaping in selectgo).
//
// selectgo returns the index of the chosen scase, which matches the
// ordinal position of its respective select{recv,send,default} call.
// Also, if the chosen scase was a receive operation, it reports whether
// a value was received.
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool) {
	......

	// generate permuted order
    for i := 1; i < ncases; i++ {
        j := fastrandn(uint32(i + 1))
        pollorder[i] = pollorder[j]
        pollorder[j] = uint16(i)
    }
    ......
    
     // sort the cases by Hchan address to get the locking order.
    // simple heap sort, to guarantee n log n time and constant stack footprint.
    for i := 0; i < ncases; i++ {
        j := i
        // Start with the pollorder to permute cases on the same channel.
        c := scases[pollorder[i]].c
        for j > 0 && scases[lockorder[(j-1)/2]].c.sortkey() < c.sortkey() {
            k := (j - 1) / 2
            lockorder[j] = lockorder[k]
            j = k
        }
        lockorder[j] = pollorder[i]
    }
    .......

    // pass 1 - look for something already waiting
    var dfli int
    var dfl *scase
    var casi int
    var cas *scase
    var recvOK bool
    for i := 0; i < ncases; i++ {
    	......
    }
    ......

    // pass 2 - enqueue on all chans
    gp = getg()
    if gp.waiting != nil {
        throw("gp.waiting != nil")
    }
    nextp = &gp.waiting
    for _, casei := range lockorder {
    	......
    }
    ......


    // wait for someone to wake us up
    gp.param = nil
    gopark(selparkcommit, nil, waitReasonSelect, traceEvGoBlockSelect, 1)

    sellock(scases, lockorder)

    gp.selectDone = 0
    sg = (*sudog)(gp.param)
    gp.param = nil

    // pass 3 - dequeue from unsuccessful chans
    // otherwise they stack up on quiet channels
    // record the successful case, if any.
    // We singly-linked up the SudoGs in lock order.
    casi = -1
    cas = nil
    sglist = gp.waiting
    // Clear all elem before unlinking from gp.waiting.
    for sg1 := gp.waiting; sg1 != nil; sg1 = sg1.waitlink {
        sg1.isSelect = false
        sg1.elem = nil
        sg1.c = nil
    }
    gp.waiting = nil

    for _, casei := range lockorder {
    	......
    }
    ......

retc:
    if cas.releasetime > 0 {
        blockevent(cas.releasetime-t0, 1)
    }
    return casi, recvOK

sclose:
    // send on closed channel
    selunlock(scases, lockorder)
    panic(plainError("send on closed channel"))
}

```

如上是对golang中select实现的梳理，可以参考。

> go版本为: go version go1.12.12 darwin/amd64
