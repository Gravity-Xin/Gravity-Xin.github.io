---
title: Golang-tcp
date: 2018-12-15 11:35:04
categories:
- Golang
tags:
- Golang
- tcp
comments: true
---

Golang的net库为了配合runtime对协程进行调度，使用阻塞和唤醒goroutine的方式对操作系统提供的网络IO接口进行了封装，用户只需要使用Block IO的形式在一个goroutine里面对一个网络连接进行读写即可，大大降低了开发者的心智负担。

一个Golang Server的典型形式:

```Go
func main(){
    l, err := net.Listen("tcp", ":8080") // 监听
    if err != nil{
        return
    }
    for{
        conn, err := net.Accept() // 建立连接
        if err != nil{
            break
        }
        go handle(conn) // 在一个goroutine中处理读写
    }
}

func handle(conn net.Conn){
    defer conn.Close()
    for{
        // 基于conn进行读写
    }
}
```

一个Golang Client的典型形式:

```Go
func main(){
    conn, err := net.Dial("tcp", "www.baidu.com:80") // 也可以使用DialTimeout接口，增加了超时机制
    if err != nil{
        return
    }
    // 基于conn进行读写
}
```

## 基本数据结构

```Go
// Listener与TCPListener 监听器
type Listener interface{
    Accept()(c Conn, err error)
    Close() error
    Addr() Addr
}

type TCPListener struct{
    fd *netFD
}

//Conn与TCPConn 网络连接
type Conn interface {
    Read(b []byte) (n int, err error) // 读取

    Write(b []byte) (n int, err error) //写入

    Close() error //关闭

    LocalAddr() Addr //本方地址

    RemoteAddr() Addr //对方地址

    SetDeadline(t time.Time) error //设置读写超时时间

    SetReadDeadline(t time.Time) error

    SetWriteDeadline(t time.Time) error
}

type TCPConn struct {
    conn
}

type conn struct {
    fd *netFD
}

//netFD的成员
type FD struct {
    fdmu fdMutex // 对读写方法进行保护，使得Conn可以跨越多个goroutine使用

    Sysfd int // 操作系统的FD值，不可变

    pd pollDesc // 对操作系统IO进行封装，I/O轮询器

    iovecs *[]syscall.Iovec // 写缓存

    csema uint32 // 关闭FD会被触发对信号量

    isBlocking uint32 //FD是否被设置为Blocking mode

    IsStream bool // 是否被设置为streaming，与其对应的是UDP的packet

    ZeroReadIsEOF bool // 读取到ZeroByte是否表示EOF

    isFile bool // FD对应的是一个文件还是一个socket
}
```

## 建立连接

  Dial、Listen

## 进行读写

  使用Conn对象进行读写，同时可以设置读写超时时间
  读取数据时，如果没有可读数据则会阻塞；如果有部分数据则直接读取完毕，并且可以获取读取的数据长度
  写入数据时，由于发送双方存在协议栈的数据缓冲区，当其满时，写入操作则会阻塞
  可以使用bufio、io/util包对Conn进行包装

  通过类型断言，可以将Conn转换为TCPConn，然后设置其属性

## 关闭连接

  Conn.Close