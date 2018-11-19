---
title: Golang-优雅关闭Channel
date: 2018-11-19 14:17:46
categories:
- Golang
tags:
- Golang
- channel
comments: true
---

## channel的特性

* 缓冲与非缓冲
* 通过select从多个channel中读取
* 通过range对一个channel进行循环读取
* 从nil channel读写会阻塞
* 从被close的channel写会panic，读会返回(零值, false)
* 已经被close的channel，再次close会panic
  
关闭channel的原则: **不要在消费端关闭，不要关闭有多个并行的生产者的channel**

## 暴力解决方案

使用recover机制来处理channel的关闭

* 生产者的Send操作

```Go
func SafeSend(ch chan T, value T)(closed bool){
    defer func(){
        if recover() != nil{
            closed = true
        }
    }
    ch <- value
    close = false
    return
}
```

* 生产者的close操作

```Go
func SafeClose(ch chan T)(justClosed bool){
    defer func(){
        if recover() != nil{
            justClosed = false
        }
    }
    close(ch)
    justClosed = true
    return
}
```

当然，也可以使用sync.Once或者sync.Mutex来执行close操作

## 优雅解决方案

上述的SafeSend()存在一个缺陷，不能使用select+case语句进行Send操作

* 多个消费者，单个生产者的情况

```Go
func main() {
    ch := make(chan int)

    go Sender(ch)
    for i := 0; i < 5; i++ {
        go Receiver(ch)
    }

    time.Sleep(5 * time.Second)
}

//Sender 生产者
func Sender(ch chan<- int) {
    maxRandomNumber := 100
    rand.Seed(time.Now().UnixNano())
    for {
        if value := rand.Intn(maxRandomNumber); value == 0 {
            close(ch) //单个生产者的情况下，直接close即可
            fmt.Println("Channel Close")
            return
        } else {
            fmt.Println("Sender: ", value)
            ch <- value
        }
    }
}

//Receiver 消费者
func Receiver(ch <-chan int) {
    for value := range ch { //对被close的channel进行range，直接退出循环
        fmt.Println("Receiver: ", value)
    }
}
```

* 多个生产者，一个消费者的情况

```Go
func main() {
    ch := make(chan int)
    stopChan := make(chan interface{}) //通过额外的Signal Channel，让Receiver通知Sender停止发送

    go Receiver(ch, stopChan)

    for i := 0; i < 5; i++ {
        go Sender(ch, stopChan)
    }

    time.Sleep(5 * time.Second)
}

//Sender 生产者
func Sender(ch chan<- int, stopChan <-chan interface{}) {
    maxRandomNumber := 100
    rand.Seed(time.Now().UnixNano())
    for {
        value := rand.Intn(maxRandomNumber)
        select {
        case <-stopChan: //接收到Stop Channel，停止发送
            return
        case ch <- value:
            fmt.Println("Sender: ", value)
        }
    }
}

//Receiver 消费者
func Receiver(ch <-chan int, stopChan chan<- interface{}) {
    for value := range ch {
        if value == 0 {
            close(stopChan)
            fmt.Println("Close Stop Channel") //通知Sender停止发送
        }
        fmt.Println("Receiver: ", value)
    }
}
```

* 多个生产者，多个消费者
    通过引入一个第三方的协调者，消费者和生产者可以主动通知协调者来关闭channel

```Go
func main() {
    ch := make(chan int)
    stopChan := make(chan string)
    doStop := make(chan interface{})

    for i := 0; i < 5; i++ {
        go Receiver("receiver:"+strconv.Itoa(i), ch, stopChan, doStop)
    }

    for i := 0; i < 5; i++ {
        go Sender("sender:"+strconv.Itoa(i), ch, stopChan, doStop)
    }

    go func() {
        from := <-stopChan //使用stopChan作为一个协调者
        fmt.Println("Close From:" + from)
        close(doStop) //在此关闭doStop channel
    }()

    time.Sleep(5 * time.Second)
}

//Sender 生产者
func Sender(name string, ch chan<- int, stopChan chan<- string, doStop <-chan interface{}) {
    maxRandomNumber := 100
    rand.Seed(time.Now().UnixNano())
    for {
        value := rand.Intn(maxRandomNumber)
        if value == 0 {
            select {
            case stopChan <- name: //通知协调者关闭doStop channel，此处采用的是select，因为Sender和Receiver都有可能向stopChan中写
                fmt.Println(name + " Active Exit")
                return //当前goroutine直接退出
            default:
                fmt.Println(name + " Active Exit")
                return
            }
        }
        select {
        case <-doStop: //其他goroutine通知协调者关闭doStop，在此退出
            fmt.Println(name + " InActive Exit")
            return
        case ch <- value:
            fmt.Println(name, value)
        }
    }
}

//Receiver 消费者
func Receiver(name string, ch <-chan int, stopChan chan<- string, doStop <-chan interface{}) {
    for {
        select {
        case <-doStop: //其他goroutine通知协调者关闭doStop，在此退出
            fmt.Println(name + " InActive Exit")
            return
        case value := <-ch:
            if value == 99 {
                select {
                case stopChan <- name: //通知协调者关闭doStop channel，此处采用的是select，因为Sender和Receiver都有可能向stopChan中写
                    fmt.Println(name + " Active Exit")
                    return
                default:
                    fmt.Println(name + " Active Exit")
                    return
                }
            }
            fmt.Println(name, value)
        }
    }
}
```