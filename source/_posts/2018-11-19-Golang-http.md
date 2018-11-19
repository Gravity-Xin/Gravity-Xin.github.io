---
title: Golang-http
date: 2018-11-19 15:33:53
categories:
- Golang
tags:
- Golang
- http
- router
comments: true
---

net/http包提供了HTTP客户端和服务端的实现

* 客户端
  * http.Request http请求
  * http.Client http客户端
  * http.Transport 代理、TLS配置、Keep-Alive、压缩等设置

* 服务端
  * http.Server http服务
  * http.DefaultServeMux http服务默认的多路复用处理器
  * http.Handle和http.HandleFunc 向DefaultServeMux添加处理函数

## 常量

DefaultMaxHeaderBytes HTTP请求头的默认最大大小，默认1MB
DefaultMaxIdleConnsPerHost 默认的持久连接设置，HTTP请求对应每个服务的持久连接数，默认2
TimeFormat HTTP头中的时间格式

## 对象

Client
    HTTP客户端
    内部缓存了TCP连接(持久连接)，因此需要尽量复用该对象
    管理HTTP的Cookie和Header信息在进行重定向时的细节

```Go
type Client struct {
    //执行独立、单次HTTP请求的机制
    Transport RoundTripper

    //设置处理重定向的策略
    //req和via分别是将要执行的请求和已经执行的请求
    CheckRedirect func(req *Request, via []*Request) error

    //Cookie管理器
    Jar CookieJar

    //请求的超时时间，包括连接时间、重定向和读取响应的时间
    Timeout time.Duration
}
```

Transport
    实现了RoundTripper接口
    支持HTTP、HTTPs、HTTP代理(基于CONNECT)
    默认缓存TCP连接，可以通过CloseIdleConnections()、MaxIdleConnsPerHost和DisableKeepAlives对缓存的连接进行控制

```Go
type Transport struct {
    idleMu     sync.Mutex
    wantIdle   bool                                // user has requested to close all idle conns
    idleConn   map[connectMethodKey][]*persistConn // most recently used at end
    idleConnCh map[connectMethodKey]chan *persistConn
    idleLRU    connLRU

    reqMu       sync.Mutex
    reqCanceler map[*Request]func(error)

    altMu    sync.Mutex   // guards changing altProto only
    altProto atomic.Value // of nil or map[string]RoundTripper, key is URI scheme

    //指定一个对给定请求返回URL代理的函数
    Proxy func(*Request) (*url.URL, error)

    //指定创建TCP连接的拨号函数，Dial已被废弃
    DialContext func(ctx context.Context, network, addr string) (net.Conn, error)
    Dial func(network, addr string) (net.Conn, error)

    //TLS的拨号函数
    DialTLS func(network, addr string) (net.Conn, error)

    //TLS配置
    TLSClientConfig *tls.Config

    //TLS握手超时时间
    TLSHandshakeTimeout time.Duration

    //是否使用持久连接，即对TCP连接进行缓存和复用
    DisableKeepAlives bool

    //是否禁止压缩
    DisableCompression bool

    //所有主机下的最大闲置TCP连接数
    MaxIdleConns int

    //每个主机下的最大闲置TCP连接数
    MaxIdleConnsPerHost int

    //所有主机下的TCP连接最大空闲时间
    IdleConnTimeout time.Duration

    //等待HTTP响应的超时时间，并不包含响应的读取时间
    ResponseHeaderTimeout time.Duration

    //等待回复HTTP头域的超时时间
    ExpectContinueTimeout time.Duration

    TLSNextProto map[string]func(authority string, c *tls.Conn) RoundTripper

    //指定在通过CONNECT处理代理时要发送的HTTP头
    ProxyConnectHeader Header

    //HTTP响应中头部数据的最大值
    MaxResponseHeaderBytes int64

    nextProtoOnce sync.Once
    h2transport   *http2Transport // non-nil if http2 wired up
}
```

Header
    HTTP头部的键值对

```Go
type Header map[string][]string
```

Cookie
    HTTP Cookie信息

```Go
type Cookie struct {
    Name  string
    Value string

    Path       string    // 路径
    Domain     string    // 域名
    Expires    time.Time // 过期时间
    RawExpires string    // for reading cookies only

    // MaxAge=0 means no 'Max-Age' attribute specified.
    // MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
    // MaxAge>0 means Max-Age attribute present and given in seconds
    MaxAge   int
    Secure   bool
    HttpOnly bool
    SameSite SameSite // Go 1.11
    Raw      string
    Unparsed []string // Raw text of unparsed attribute-value pairs
}
```

Request
    表示客户端发送出去或者服务端收到的HTTP请求
    在客户端和服务端中可能会有不同的语义

```Go
type Request struct {
    Method string //请求方法

    URL *url.URL //请求URL

    Proto      string // HTTP协议 HTTP/1.1 HTTP/2. HTTP/1.0
    ProtoMajor int    // 1
    ProtoMinor int    // 0

    Header Header // 请求头部信息

    //请求体
    //客户端: Client的Transport会关闭该对象
    //服务端: Server会关闭该对象
    Body io.ReadCloser

    GetBody func() (io.ReadCloser, error)

    ContentLength int64 //Body的长度

    TransferEncoding []string

    //客户端: 是否在发送请求后关闭该连接
    //服务端: 是否在回复请求后关闭连接
    Close bool

    //主机名
    Host string

    //通过ParseForm来生成该对象
    //包含Get请求的Query参数和Post、Put请求的表单数据
    Form url.Values

    //通过ParseForm来生成该对象
    //包含Post、Put请求的表单数据
    PostForm url.Values

    //通过ParseMultipartForm来生成该对象
    //包含多部件表单数据
    MultipartForm *multipart.Form

    Trailer Header

    //"IP:port"
    RemoteAddr string

    //客户端发送到服务端的请求的请求行中未修改的请求URI
    RequestURI string

    //TLS连接信息
    TLS *tls.ConnectionState

    //已废弃，使用Context来实现取消或超时机制
    Cancel <-chan struct{}

    //重定向情况下，产生该Request对应的Response
    Response *Response

    ctx context.Context
}
```

Handler
    HTTP请求的处理器
    实现了该接口的对象可以被注册的Server的ServerMux中，为特定的URL路径提供服务
    FileServer()、StripPrefix()、NotFoundHandler()、RedirectHandler()、TimeoutHandler()分别提供了文件服务、去除URL前缀、NotFound、重定向和超时的处理器

```Go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

HandlerFunc
    实现将普通函数转换为Handler的适配器

```Go
type HandlerFunc func(ResponseWriter, *Request) //将函数定义为对象，实现普通函数与Handler接口之间的转换

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r) //在这里调用函数本身
}
```

ServeMux
    HTTP请求的多路复用器，将URL和对应的Handler进行关联(路由)
    实现了Handler接口

```Go
type ServeMux struct {
    mu    sync.RWMutex
    m     map[string]muxEntry
    hosts bool // whether any patterns contain hostnames
}
```

Server
    HTTP服务

```Go
type Server struct {
    Addr    string  // 监听的TCP地址
    Handler Handler // 请求的处理器，默认使用DefaultServeMux

    TLSConfig *tls.Config //TLS配置

    ReadTimeout time.Duration //读取请求的超时时间

    ReadHeaderTimeout time.Duration //读取请求中Header的超时时间

    WriteTimeout time.Duration //写响应的超时时间

    IdleTimeout time.Duration //Keep-Alive机制下，连接的最大空闲时间

    MaxHeaderBytes int //请求头域的最大长度

    TLSNextProto map[string]func(*Server, *tls.Conn, Handler)

    ConnState func(net.Conn, ConnState) //与客户端的连接状态发生改变时的回调函数

    ErrorLog *log.Logger //日志记录器

    disableKeepAlives int32     // accessed atomically.
    inShutdown        int32     // accessed atomically (non-zero means we're in Shutdown)
    nextProtoOnce     sync.Once // guards setupHTTP2_* init
    nextProtoErr      error     // result of http2.ConfigureServer if used

    mu         sync.Mutex
    listeners  map[net.Listener]struct{}
    activeConn map[*conn]struct{}
    doneChan   chan struct{}
    onShutdown []func()
}
```

ResponseWriter
    HTTP响应的接口，用于在Handler中写入HTTP响应

```Go
type ResponseWriter interface {
    Header() Header //返回HTTP响应头域

    Write([]byte) (int, error) //写入HTTP响应的Body

    WriteHeader(statusCode int) //写入HTTP响应的状态码
}
```

Response
    HTTP响应

```Go
type Response struct {
    Status     string // e.g. "200 OK"
    StatusCode int    // e.g. 200
    Proto      string // e.g. "HTTP/1.0"
    ProtoMajor int    // e.g. 1
    ProtoMinor int    // e.g. 0

    Header Header //响应头域

    Body io.ReadCloser //响应体

    ContentLength int64 //Body的长度

    TransferEncoding []string

    Close bool

    Uncompressed bool //响应内容是否进行了压缩

    Trailer Header

    Request *Request //响应对应的请求

    TLS *tls.ConnectionState
}
```