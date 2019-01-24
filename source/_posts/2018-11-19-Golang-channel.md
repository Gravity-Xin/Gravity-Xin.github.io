---
title: Golang-channel
date: 2018-11-19 14:17:46
categories:
- Golang
tags:
- Golang
- channel
comments: true
---

## channel的特性

* 类型
  * 缓冲(len, cap操作获取状态)
  * 非缓冲

* 状态
  * nil，未初始化的状态，只进行了声明
  * active，正常的状态，可以进行读写
  * close，已关闭的状态，但是不为nil

* 操作
  * 读与写
  * 关闭
    * channel并不需要显式的close，如果没有goroutine对其读写，则channel会被GC回收

![Channel的状态与操作](/images/Golang之channel的状态与操作.png)

特殊情况: 当nil channel在select的某个case中时，这个case会阻塞，但不会造成panic

## channel的常用操作

* 使用`for range`循环读取channel

当channel关闭时，for循环会自动退出，无需主动监测channel是否关闭，可以防止读取已经关闭的channel，造成读到数据为通道所存储的数据类型的零值

* 使用`_, ok`判断channel是否关闭

读已关闭的channel会造成零值 ，如果不确定channel，需要使用ok进行检测。ok为`true`表示读到数据，并且通道没有关闭; ok为`false`表示通道关闭

* 使用`select`处理多个channel的读写

同时监控多个channel的情况，只处理最先未阻塞的case; 当通道为nil时，对应的case永远为阻塞，无论读写
普通情况下，对nil的通道写操作是要panic的

* 使用channel的声明控制读写权限

如果协程对某个channel只有写操作，则这个channel声明为只写
如果协程对某个channel只有读操作，则这个channe声明为只读

* 使用缓冲channel增强并发和异步

* 使用`select`和`time.After`为操作加上超时

看操作和定时器哪个先返回，处理先完成的，达到超时控制的效果

当该操作为channel的读写时，可以为channel操作增加超时效果，使得channel的读写变为无阻塞

* 使用`close`关闭channel,从而关闭channel所有的下游协程

* 使用chan struct{}作为信号channel

当数据需要传递时，传递空struct

* 使用channel传递结构体的指针而非结构体

channel本质上传递的是数据的拷贝，拷贝的数据越小传输效率越高，传递结构体指针，比传递结构体更高效

## 优雅关闭channel

关闭channel的原则: **不要在消费端关闭，不要关闭有多个并行的生产者的channel**

### 暴力解决方案

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

### 优雅解决方案

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