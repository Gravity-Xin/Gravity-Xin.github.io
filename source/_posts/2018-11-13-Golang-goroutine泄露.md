---
title: Golang-goroutine泄露
date: 2018-11-13 16:59:31
categories:
- Golang
tags:
- Golang
- Goroutine
comments: true
---

本文参考[Goroutine leak](https://medium.com/golangspec/goroutine-leak-400063aef468)

## 何为Goroutine泄露

Go的并发是通过goroutine和channel实现的。由于goroutine的存在会消耗内存等资源，如果goroutine随着程序的运行不断的增长，最终会耗尽内存，导致程序崩溃。

## 产生泄露的情况

* 向没有receiver的channel发送消息

```Go
// 模拟查询请求
func query() int {  
    n := rand.Intn(100)  
    time.Sleep(time.Duration(n) * time.Millisecond)  
    return n  
}

// 发起多个请求，然后获取最先得到的结果
func queryAll() int {  
    ch := make(chan int)  
    go func() { ch <- query() }()  
    go func() { ch <- query() }()  
    go func() { ch <- query() }()  
    return <-ch  
}

func main() {  
    for i := 0; i < 4; i++ {  
        queryAll()  
        fmt.Printf("#goroutines: %d\n", runtime.NumGoroutine())  
    }  
}
```

```shell
#goroutines: 3  
#goroutines: 5  
#goroutines: 7  
#goroutines: 9
```

同时发起多个请求，然后获取最先得到结果的响应，此时较慢的goroutine将会出现向没有接收者的channel发送消息的情况

可能的解决方法是，如果知道请求数量，则可以使用带缓冲的channel，否则可以在匿名方法中使用context的超时机制

* 从没有sender的channel获取消息
  
原理同第一种情况

* 使用了值为nil的channel

对值为nil的channel进行读写会永远阻塞

* 程序陷入了循环

goroutine陷入了死循环，导致永远不会结束

## 检查方法

* 使用runtime.NumGoroutine()来观察

* 对于Web应用，使用net/http/pprof，通过/debug/pprof/goroutine?debug=1来观察

* 使用runtime/pprof将现有的goroutine的堆栈打印

```Go
pprof.Lookup("goroutine").WriteTo(os.Stdout, 1)
```

* 使用[gops](github.com/google/gops)

* 使用[leaktest](https://github.com/fortytw2/leaktest)
