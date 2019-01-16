---
title: Golang-实现简单的https服务器
date: 2019-01-16 17:05:47
categories:
- Golang
tags:
- https
comments: true
---

## TLS协议

https协议使用TLS(Transport Layer Security, 传输层安全协议)来保证通信的安全性和数据的完整性，该协议工作在TCP协议之上，可以看做是SSL协议的升级版

TLS基于X.509来进行认证，然后使用非对称加密算法来进行通信双方的身份认证，然后进行公钥的交换，并使用公钥对通信双方的数据进行加密

## 生成证书

* 生成服务端证书

```shell
# 生成私钥
openssl genrsa -out server.key 2048
# 生成证书
openssl req -new -x509 -key server.key -out server.pem -days 3650
```

或者直接使用go内置工具

```shell
go run $GOROOT/src/crypto/tls/generate_cert.go --host localhost
```

* 生成客户端证书

并不是必须的
但在某些需要验证客户端身份的场景下，服务器需要客户端提供证书

```shell
# 生成私钥
openssl genrsa -out client.key 2048
# 生成证书
openssl req -new -x509 -key client.key -out client.pem -days 3650
```

## 证书的使用

golang的`crypto/tls`实现了TLS1.2的功能，`crypto/x509`实现了证书管理的功能

* TCP服务器

```Go
func main() {
    cert, err := tls.LoadX509KeyPair("server.pem", "server.key") // 获取服务端证书
    if err != nil {
        log.Println(err)
        return
    }
    certBytes, err := ioutil.ReadFile("client.pem") // 读取客户端证书
    if err != nil {
        panic("Unable to read cert.pem")
    }
    clientCertPool := x509.NewCertPool()
    ok := clientCertPool.AppendCertsFromPEM(certBytes)
    if !ok {
        panic("failed to parse root certificate")
    }
    config := &tls.Config{
        Certificates: []tls.Certificate{cert},
        ClientAuth:   tls.RequireAndVerifyClientCert, // 设置需要对客户端身份进行认证
        ClientCAs:    clientCertPool, // 添加客户端证书池
    }
    config := &tls.Config{Certificates: []tls.Certificate{cert}} // 创建tls配置
    ln, err := tls.Listen("tcp", ":443", config) // 开启监听
    if err != nil {
        log.Println(err)
        return
    }
    defer ln.Close()
    for {
        conn, err := ln.Accept() // 建立连接
        if err != nil {
            log.Println(err)
            continue
        }
        go handleConn(conn) // 处理请求
    }
}

func handleConn(conn net.Conn) {
    defer conn.Close()
    r := bufio.NewReader(conn)
    for {
        msg, err := r.ReadString('\n')
        if err != nil {
            log.Println(err)
            return
        }
        log.Println(msg)
        n, err := conn.Write([]byte("world\n"))
        if err != nil {
            log.Println(n, err)
            return
        }
    }
}
```

* TCP客户端

```Go
func main() {
    cert, err := tls.LoadX509KeyPair("client.pem", "client.key")
    if err != nil {
        log.Println(err)
        return
    }
    certBytes, err := ioutil.ReadFile("client.pem")
    if err != nil {
        panic("Unable to read cert.pem")
    }
    clientCertPool := x509.NewCertPool()
    ok := clientCertPool.AppendCertsFromPEM(certBytes)
    if !ok {
        panic("failed to parse root certificate")
    }
    conf := &tls.Config{
        RootCAs:            clientCertPool,
        Certificates:       []tls.Certificate{cert}, // 在tls配置中添加客户端证书信息
        InsecureSkipVerify: true, // 设置为true，则不会校验服务器证书以及证书中的主机名和服务器主机名是否一致
    }
    conn, err := tls.Dial("tcp", "127.0.0.1:443", conf) // 建立连接
    if err != nil {
        log.Println(err)
        return
    }
    defer conn.Close()
    n, err := conn.Write([]byte("hello\n")) // 写数据
    if err != nil {
        log.Println(n, err)
        return
    }
    buf := make([]byte, 100)
    n, err = conn.Read(buf) // 读数据
    if err != nil {
        log.Println(n, err)
        return
    }
    log.Println(string(buf[:n]))
}
```

* HTTPs服务器

```Go
func main() {
    http.HandleFunc("/hello", helloServerfunc)
    server := &http.Server{
        Addr: ":443",
        TLSConfig: &tls.Config{
            ClientAuth: tls.RequestClientCert, // 配置需要对客户端证书进行验证
        },
    }
    err := server.ListenAndServeTLS("cert/server.pem", "cert/server.key")
    if err != nil {
        log.Fatal(err)
    }
}

func helloServerfunc(rw http.ResponseWriter, r *http.Request) {
    rw.Header().Set("Content-Type", "text/plain")
    rw.Write([]byte("This is an example server.\n"))
}
```

在浏览器中输入`https://localhost/hello`即可访问