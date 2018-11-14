---
title: Golang-goroutine的调度
date: 2018-11-14 09:58:27
categories:
- Golang
tags:
- Golang
- goroutine
comments: true
---

并发与并行
并行指的是在不同的物理处理器上同时执行不同的代码片段，并行可以同时做很多事情，而并发是同时管理很多事情

全局运行队列	所有刚创建的goroutine都会放到这里
本地运行队列	逻辑处理器的goroutine队列
当我们创建一个goroutine的后，会先存放在全局运行队列中，等待Go运行时的调度器进行调度，把他们分配给其中的一个逻辑处理器，并放到这个逻辑处理器对应的本地运行队列中，最终等着被逻辑处理器执行即可。