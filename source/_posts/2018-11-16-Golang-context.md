---
title: Golang-context
date: 2018-11-16 13:00:04
categories:
- Golang
tags:
- Golang
- context
comments: true
---

## 背景

在Go编写的服务器程序中，多个请求在单独的goroutine中运行，Context为这些请求增加了`截止日期(请求超时)、主动取消请求、存储和请求相关的数据`的功能，并且是多goroutine下可以安全的使用

## Context接口

```Go
type Context interface {
    Deadline() (deadline time.Time, ok bool) //获取Context的截止日期，如果没有设置截止日期，则ok==false

    Done() <-chan struct{} // 返回Context关闭时会被close的chan，这个chan可在多个goroutine上进行广播，在截止日期到达、主动关闭时Context会被关闭
                           // 一般用在select语句中

    Err() error // 返回Context关闭的原因，如果Context尚未被关闭，则返回nil

    Value(key interface{}) interface{} // 返回Context中存储的值，可以通过类型推断来获取值的具体类型
}
```

## 基本方法

多个Context之间可以有继承关系

`Background`: 所有Context的root，没有截止日期，也不会被关闭
`WithCancel`: 返回一个可以主动取消的Context，当父Context的Done被关闭或者主动取消时，该Context的Done会被关闭
`WithDeadline`, `WithTimeout`: 返回一个可以主动取消和设置了截止时间的Context，当父Context的Done被关闭或者主动取消或到达截止时间时，该Context的Done会被关闭
`WithValue`: 返回一个包含了K-V对的Context

## 示例

主要关闭的Context

```Go
func someHandler() {
    ctx, cancel := context.WithCancel(context.Background())
    go doStuff(ctx)
    time.Sleep(10 * time.Second) //10秒后取消
    cancel()
}

func doStuff(ctx context.Context) {
    for {
        time.Sleep(1 * time.Second)
        select { //每1秒work一下，同时会判断ctx是否被取消了，如果是就退出
        case <-ctx.Done():
            logg.Printf("done")
            return
        default:
            logg.Printf("work")
        }
    }
}

```