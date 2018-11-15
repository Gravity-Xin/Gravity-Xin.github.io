---
title: Golang-runtime
date: 2018-11-14 09:57:07
categories:
- Golang
tags:
- Golang
- runtime
- goroutine
- heap
comments: true
---

runtime提供了goroutine的运行时环境。
下述的环境变量控制了runtime的行为，主要包括:

``` shell
GOGC
  控制GC的频率，当自上次GC后HeapSize已经增长了GOGC%后触发GC
  默认为100，即HeapSize增加一倍后触发GC；GOGC=off则关闭GC
  可以通过runtime/debug.SetGCPercent函数在运行时修改GOGC值
GODEBUG
  控制runtime的debug输出，值为name1=value1,name2=value2的形式
  支持的name如下
    allocfreetrace: 设置1会记录每个对象的内存分配和释放的堆栈踪迹
    cgocheck: 设置0会不再检查go指针在非go代码中的使用情况
    efence: 设置1会导致内存分配器的模式变为对每个对象使用独立的页和地址且永不循环使用
    gccheckmark: 设置为1会开启对GC标记的二次检查
    gcpacetrace:
    gcshrinkstackoff:
    gcrescanstacks:
    gcstoptheworld:
    gctrace: 设置为1会在每次GC时向标准错误输出写入单行数据，包括GC内存的大小、暂停的时间长度等
    memprofilerate: 修改runtime.MemProfileRate值
    invalidptr:
    sbrk:
    scavenge:
    scheddetail: 设置schedtrace为X并设置其为1，会导致调度器每隔X毫秒向标准错误输出写入详细的多行信息
    schedtrace: 设置为X会导致goroutine调度器每隔X毫秒向标准错误输出写入单行数据，概述调度状态
    tracebackancestors:
GOMAXPROCS
    设置同时运行goroutine的操作系统线程数
GOTRACEBACK
    控制当Go程序运行失败时的输出信息
GOARCH、GOOS、GOPATH、GOROOT
    影响Go程序的构建
```

runtime中的变量

```shell
MemProfileRate
   控制内存profile记录的内存分配的采样频率，内存profile记录器平均每分配MemProfileRate字节进行一次分配采样
```

runtime中的函数

```shell
BlockProfile()
    返回当前阻塞的goroutine的profile
SetBlockProfileRate()
    设置goroutine阻塞事件profile的采样评测，平均每阻塞多少纳秒，记录器就会采集一份样本
SetCPUProfileRate()
    设置CPU profile记录器的记录频率为每秒多少次
Caller()
    返回goroutine调用栈所执行的函数的文件和行号信息
Callers()
    把当前goroutine调用栈上的调用栈标识符填入切片中
GC()
    执行一次GC操作
GOMAXPROCS(int)
    设置GOMAXPROCS环境变量
Goexit()
    终止当前的goroutine
GoroutineProfile()
    返回当前激活的goroutine的堆栈profile
Gosched()
    使当前的goroutine放弃CPU，以让其它go程运行，它不会挂起当前goroutine，因此当前goroutine未来会恢复执行
KeepAlive()
    使得某个对象reachable，不会被回收，也就不会调用其finalizer
LockOSThread()
UnLockOSThread()
    将调用的goroutine绑定到它当前所在的操作系统线程，除非调用的go程退出或调用UnlockOSThread，否则它将总是在该线程中执行，而其它go程则不能进入该线程
MemProfile()
    返回当前内存profile中的记录数
MutexProfile()
    返回当前系统所有goroutine的block profile
SetMutexProfileFraction()
    设置mutex记录器的统计的比例
NumCPU()
    返回本地机器的逻辑CPU数量
NumCgoCall()
    返回当前系统中cgo调用数量
NumGoroutine()
    返回系统中存在的goroutine数量
ReadMemStats()
    将内存申请的统计信息保存
SetFinalizer()
    设置对象的清理函数
Stack()
    返回当前goroutine的堆栈信息
FuncForPC()
    返回一个调用栈标识符pc对应的调用栈的*Func
```

runtime中的对象

```shell
Func: 堆栈中的函数对象
MemStats: 内存分配器的统计信息
```

从上述内容可以看出，runtime提供的主要功能包括goroutine的调度和内存GC。具体包括debug和pprof进行性能分析、各个记录器来抓取系统事件信息，如goroutine创建、mutex|block、内存分配、CPU使用等等、

## goroutine的调度

* 并发与并行
  * 并行指的是在不同的物理处理器上同时执行不同的代码片段，并行可以同时做很多事情，而并发是同时管理很多事情

* 为什么需要语言级别的调度
  * OS的线程调度代价大
  * GC需要stop the workd，因此需要对goroutine进行调度，使其在GC时停止

* 主要组成(G、P、M)
  * 每一个Go程序都自带一个runtime，负责与OS进行交互
  * G: goroutine M: machine|thread
  * P: processor 可以看作是一个在单线程上运行的goroutine队列

![image](/images/goroutine调度之G、P、M.jpg)

* 调度细节
  * 全局P: 所有刚创建的goroutine放在这里
  * 本地P: 所有运行的goroutine
  * 当我们创建一个goroutine的后，会先存放在全局P中。等待Go运行时的调度器进行调度，把他们分配给其中的一个本地P
  * 处于本地P中的goroutine，依次排队等待最终等着CPU执行
  * 运行的goroutine因为系统调用而阻塞的时候，将这个goroutine放入到全局P中
  * 各个本地P在运行完自己的队列后，也可以从全局P中获取goroutine

* 特点
  * goroutine的动态栈
  * 抢占式调度，长时间对于运行状态的P会被其他的P抢占
  * G与M之间时多对多的关系

## GC

* 低延迟GC，代价是GC吞吐量的下降
* 三色标记清除