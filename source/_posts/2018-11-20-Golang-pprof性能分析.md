---
title: Golang-pprof性能分析
date: 2018-11-20 14:01:03
categories:
- 编程技术
tags:
- Golang
- pprof
- goroutine
- heap
comments: true
---

runtime包提供了采集程序运行时相关性能数据的接口，而net/http/pprof包则是对HTTP服务运行性能数据进行分析和可视化的工具

## 使用方式

* 导入"net/http/pprof"包，在该包的初始化时，会将相关HTTP Handler注册到默认ServeMux中
  * 访问`http:host:port/debug/pprof/`
    * `/debug/pprof/profile` 默认进行30s间隔采样的CPU使用情况
    * `/debug/pprof/block` 查看导致阻塞同步的堆栈跟踪
    * `/debug/pprof/goroutine` 查看当前所有运行的 goroutines 堆栈跟踪
    * `/debug/pprof/heap` 查看活动对象的内存分配情况
    * `/debug/pprof/mutex` 查看导致互斥锁的竞争持有者的堆栈跟踪
    * `/debug/pprof/threadcreate` 查看创建新OS线程的堆栈跟踪
  * 使用命令行工具`go tool pprof` + 上述可选URL
  * 使用命令行工具`go test -bench=. -cpuprofile=cpu.prof -memprofile=mem.prof`来生成报告，然后使用`go tool pprof cpu.prof`来可视化展示报告内容
  * 使用pprof工具
    * 安装 `go get -u github.com/google/pprof`
    * 启动 `pprof cpu.prof`
    * `web`命令 以图表形式展示
    * `topN`命令 展示最耗时的n个堆栈
  * 使用火焰图工具go-torch
    * 直接读取profile文件，生成火焰图的svg文件

## 可以分析的内容

* CPU Profile: CPU使用情况分析
* Memory Profile: 内存堆分配时的堆栈，检查内存泄露
* Block Profile: 协程阻塞分析，记录协程阻塞的位置
* Mutex Profile: 互斥锁分析，报告互斥锁的竞争情况