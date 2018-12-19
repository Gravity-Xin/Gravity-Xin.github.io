---
title: Golang-grpc
date: 2018-12-19 09:55:33
categories:
- Golang
tags:
- grpc
comments: true
---

## 安装

* go-grpc

```shell
go get -u -v github.com/grpc/grpc-go
mkdir -p $GOPATH/src/google.golang.org/grpc
cp -r $GOPATH/src/github.com/grpc/grpc-go/ $GOPATH/src/google.golang.org/grpc
```

由于某些go库被墙，建议手动下载之后复制到相应的目录

* protoc v3

```shell
cd /tmp
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protoc-3.6.1-osx-x86_64.zip
unzip protoc-3.6.1-osx-x86_64.zip
sudo cp bin/protoc /usr/local/bin/
```

* protoc plugin for Go

```shell
go get -u -v github.com/golang/protobuf/protoc-gen-go
```

## 定义grpc.proto文件

* proto v3 基本语法

[proto3](https://developers.google.com/protocol-buffers/docs/proto3)

* 编写proto文件

```proto
syntax = "proto3";

package sayhello;

message Request{
    int32 id = 1;
}
message Response{
    string name = 1;
}

service Greeter{
    rpc SayHello (Request) returns (Response);
}
```

* 生成pb.go文件

```shell
protoc --go_out=plugins=grpc:. sayhello.proto
```

## 服务端

```Go
type GreetSvr struct {
}

// 接口实现
func (s *GreetSvr) SayHello(ctx context.Context, req *sayhello.Request) (*sayhello.Response, error) {
    rep := sayhello.Response{}
    rep.Name = "Hello " + strconv.Itoa(int(req.GetId()))
    return &rep, nil
}

func main() {
    listen, _ := net.Listen("tcp", ":8001") // 监听
    s := grpc.NewServer() // 创建服务
    sayhello.RegisterGreeterServer(s, &GreetSvr{}) // 将接口实现注册到服务中
    s.Serve(listen)
}
```

## 客户端

```Go
func main() {
    conn, _ := grpc.Dial("127.0.0.1:8001", grpc.WithInsecure()) // 建立连接
    client := sayhello.NewGreeterClient(conn) // 创建客户
    for i := 0; i < 100; i++ {
        req := sayhello.Request{}
        req.Id = int32(i%3 + 1)
        rep, _ := client.SayHello(context.Background(), &req) // 调用接口
        fmt.Println(rep.GetName())
    }
}
```
