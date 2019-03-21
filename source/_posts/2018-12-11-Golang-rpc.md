---
title: Golang-rpc
date: 2018-12-11 19:04:40
categories:
- 编程技术
tags:
- rpc
comments: true
---

golang官方的net/rpc库提供了RPC的实现，封装了数据序列化和底层传输的细节
服务端可以注册多个服务对象，符合下列条件的对象方法便可以被远程调用

```Go
func(t *T)MethodName(reqType T1, repType *T2) error
```

* 对象类型是导出的
* 对象方法是导出的
* 方法只有两个参数，参数类型为内置类型或者可导出的类型
* 方法的第二个参数为指针类型
* 方法的返回值为error类型

方法中的第一个参数为客户端请求对象，第二个对象为服务端响应对象

客户端通过将对象名称、方法名称和请求对象传递给服务端，然后使用Do或者Go来同步或异步进行RPC调用

## Server实现

```Go
const (
    // HTTP默认使用的Path
    DefaultRPCPath   = "/_goRPC_" //RPC的Path
    DefaultDebugPath = "/debug/rpc" //debug的Path
)

// 可以导出的方法描述
type methodType struct {
    sync.Mutex
    method     reflect.Method // 方法对象
    ArgType    reflect.Type // 请求参数对象
    ReplyType  reflect.Type // 响应参数对象
    numCalls   uint
}

// 可以导出的对象描述，一个服务可以注册多个对象
type service struct {
    name   string                 // 对象名称
    rcvr   reflect.Value          // 对象的值
    typ    reflect.Type           // 对象类型
    method map[string]*methodType // 对象注册的导出方法
}

// RPC的请求头，在每一次RPC请求之前生成
type Request struct {
    ServiceMethod string   // Service.Method 指定请求对应的对象和方法
    Seq           uint64   // Client指定的序列号
    next          *Request // 下一个请求，用于形成一个请求列表
}

// RPC的响应头，在每一次RPC响应之前生成
type Response struct {
    ServiceMethod string    // Service.Method 请求对应的对象和方法
    Seq           uint64    // Client指定的序列号
    Error         string    // 出错信息
    next          *Response // 下一个响应，用于形成一个响应列表
}

// RPC服务
type Server struct {
    serviceMap sync.Map   // map[string]*service，封装了所有的对象
    reqLock    sync.Mutex
    freeReq    *Request // 请求列表
    respLock   sync.Mutex
    freeResp   *Response // 响应列表
}

// 创建一个RPC服务
func NewServer() *Server {
    return &Server{}
}

// RPC库提供的默认RPC服务对象
var DefaultServer = NewServer()

// 判断是否可以导出，即类型名称首字符是否大写
func isExported(name string) bool {
    rune, _ := utf8.DecodeRuneInString(name)
    return unicode.IsUpper(rune)
}

// 判断类型是否可以导出或内置
func isExportedOrBuiltinType(t reflect.Type) bool {
    for t.Kind() == reflect.Ptr {
        t = t.Elem()
    }
    //通过判断PkgPath是否为空，确定是否为内置类型
    return isExported(t.Name()) || t.PkgPath() == ""
}

// 服务进行对象注册，会将对象中的所有可导出方法进行注册
func (server *Server) Register(rcvr interface{}) error {
    return server.register(rcvr, "", false)
}

// 服务进行对象注册，会将对象中的所有可导出方法进行注册，同时指定对象名称
func (server *Server) RegisterName(name string, rcvr interface{}) error {
    return server.register(rcvr, name, true)
}

// 对象注册的实际实现逻辑
func (server *Server) register(rcvr interface{}, name string, useName bool) error {
    s := new(service)
    s.typ = reflect.TypeOf(rcvr)
    s.rcvr = reflect.ValueOf(rcvr)
    sname := reflect.Indirect(s.rcvr).Type().Name()
    if useName {
        sname = name
    }
    if sname == "" { // 对象名称不可为空
        s := "rpc.Register: no service name for type " + s.typ.String()
        log.Print(s)
        return errors.New(s)
    }
    if !isExported(sname) && !useName { // 对象应该可以导出
        s := "rpc.Register: type " + sname + " is not exported"
        log.Print(s)
        return errors.New(s)
    }
    s.name = sname

    s.method = suitableMethods(s.typ, true) // 获取对象所有可以导出的方法

    if len(s.method) == 0 {
        str := ""
        // 看看对象的指针是否包含可导出的方法，如果有则提示用户使用指针对象来注册
        method := suitableMethods(reflect.PtrTo(s.typ), false)
        if len(method) != 0 {
            str = "rpc.Register: type " + sname + " has no exported methods of suitable type (hint: pass a pointer to value of that type)"
        } else {
            str = "rpc.Register: type " + sname + " has no exported methods of suitable type"
        }
        log.Print(str)
        return errors.New(str)
    }

    if _, dup := server.serviceMap.LoadOrStore(sname, s); dup { // 判断对象名称是否重复注册
        return errors.New("rpc: service already defined: " + sname)
    }
    return nil
}

// 获取一个对象可以导出的方法集合
func suitableMethods(typ reflect.Type, reportErr bool) map[string]*methodType {
    methods := make(map[string]*methodType)
    for m := 0; m < typ.NumMethod(); m++ {
        method := typ.Method(m)
        mtype := method.Type
        mname := method.Name
        if method.PkgPath != "" { // 判断方法是否可以导出
            continue
        }
        if mtype.NumIn() != 3 { // 判断方法的形参数量应该为3个，第一个为对象Receiver
            if reportErr {
                log.Printf("rpc.Register: method %q has %d input parameters; needs exactly three\n", mname, mtype.NumIn())
            }
            continue
        }
        argType := mtype.In(1) // 第一个形参不可以为指针，且参数类型可以导出
        if !isExportedOrBuiltinType(argType) {
            if reportErr {
                log.Printf("rpc.Register: argument type of method %q is not exported: %q\n", mname, argType)
            }
            continue
        }
        replyType := mtype.In(2) // 第二个形参需要为指针，且参数类型可以导出
        if replyType.Kind() != reflect.Ptr {
            if reportErr {
                log.Printf("rpc.Register: reply type of method %q is not a pointer: %q\n", mname, replyType)
            }
            continue
        }
        if !isExportedOrBuiltinType(replyType) {
            if reportErr {
                log.Printf("rpc.Register: reply type of method %q is not exported: %q\n", mname, replyType)
            }
            continue
        }
        if mtype.NumOut() != 1 { // 返回参数只有一个，且为error类型
            if reportErr {
                log.Printf("rpc.Register: method %q has %d output parameters; needs exactly one\n", mname, mtype.NumOut())
            }
            continue
        }
        if returnType := mtype.Out(0); returnType != typeOfError {
            if reportErr {
                log.Printf("rpc.Register: return type of method %q is %q, must be error\n", mname, returnType)
            }
            continue
        }
        methods[mname] = &methodType{method: method, ArgType: argType, ReplyType: replyType}
    }
    return methods
}

// 当用户请求无效时，向用户返回该对象
var invalidRequest = struct{}{}

// 使用指定的序列化对象来向客户端发送响应
func (server *Server) sendResponse(sending *sync.Mutex, req *Request, reply interface{}, codec ServerCodec, errmsg string) {
    resp := server.getResponse()
    resp.ServiceMethod = req.ServiceMethod // 响应头
    if errmsg != "" {
        resp.Error = errmsg
        reply = invalidRequest
    }
    resp.Seq = req.Seq
    sending.Lock()
    err := codec.WriteResponse(resp, reply) // 使用Codec发送响应
    if debugLog && err != nil {
        log.Println("rpc: writing response:", err)
    }
    sending.Unlock()
    server.freeResponse(resp)
}

func (m *methodType) NumCalls() (n uint) {
    m.Lock()
    n = m.numCalls
    m.Unlock()
    return n
}

// 执行具体的方法调用
func (s *service) call(server *Server, sending *sync.Mutex, wg *sync.WaitGroup, mtype *methodType, req *Request, argv, replyv reflect.Value, codec ServerCodec) {
    if wg != nil {
        defer wg.Done()
    }
    mtype.Lock()
    mtype.numCalls++
    mtype.Unlock()
    function := mtype.method.Func
    returnValues := function.Call([]reflect.Value{s.rcvr, argv, replyv}) // 调用具体的方法
    errInter := returnValues[0].Interface() // 获取方法返回的error对象
    errmsg := ""
    if errInter != nil {
        errmsg = errInter.(error).Error()
    }
    server.sendResponse(sending, req, replyv.Interface(), codec, errmsg)
    server.freeRequest(req)
}

// gob序列化实现
type gobServerCodec struct {
    rwc    io.ReadWriteCloser
    dec    *gob.Decoder
    enc    *gob.Encoder
    encBuf *bufio.Writer
    closed bool
}

func (c *gobServerCodec) ReadRequestHeader(r *Request) error {
    return c.dec.Decode(r)
}

func (c *gobServerCodec) ReadRequestBody(body interface{}) error {
    return c.dec.Decode(body)
}

func (c *gobServerCodec) WriteResponse(r *Response, body interface{}) (err error) {
    if err = c.enc.Encode(r); err != nil {
        if c.encBuf.Flush() == nil {
            log.Println("rpc: gob error encoding response:", err)
            c.Close()
        }
        return
    }
    if err = c.enc.Encode(body); err != nil {
        if c.encBuf.Flush() == nil {
            log.Println("rpc: gob error encoding body:", err)
            c.Close()
        }
        return
    }
    return c.encBuf.Flush()
}

func (c *gobServerCodec) Close() error {
    if c.closed {
        return nil // 保证Close的幂等性
    }
    c.closed = true
    return c.rwc.Close()
}

// 对一个网络请求进行处理，默认使用gob的序列化方式
func (server *Server) ServeConn(conn io.ReadWriteCloser) {
    buf := bufio.NewWriter(conn)
    srv := &gobServerCodec{
        rwc:    conn,
        dec:    gob.NewDecoder(conn),
        enc:    gob.NewEncoder(buf),
        encBuf: buf,
    }
    server.ServeCodec(srv)
}

// 与ServerConn功能相同，只是指定了序列化方式
// 完成读取请求，异步调用方法，返回结果的功能
func (server *Server) ServeCodec(codec ServerCodec) {
    sending := new(sync.Mutex)
    wg := new(sync.WaitGroup) // 使用wg来管理请求列表
    for {
        service, mtype, req, argv, replyv, keepReading, err := server.readRequest(codec) // 读取请求
        if err != nil {
            if debugLog && err != io.EOF {
                log.Println("rpc:", err)
            }
            if !keepReading {
                break
            }
            if req != nil {
                server.sendResponse(sending, req, invalidRequest, codec, err.Error())
                server.freeRequest(req)
            }
            continue
        }
        wg.Add(1)
        go service.call(server, sending, wg, mtype, req, argv, replyv, codec) // 异步调用方法
    }
    wg.Wait() // 等待所有请求已经处理完成
    codec.Close()
}

// 与ServeCodec相同，但是进行的是同步调用，在处理完单个请求之后不会关闭Codec
func (server *Server) ServeRequest(codec ServerCodec) error {
    sending := new(sync.Mutex)
    service, mtype, req, argv, replyv, keepReading, err := server.readRequest(codec)
    if err != nil {
        if !keepReading {
            return err
        }
        if req != nil {
            server.sendResponse(sending, req, invalidRequest, codec, err.Error())
            server.freeRequest(req)
        }
        return err
    }
    service.call(server, sending, nil, mtype, req, argv, replyv, codec)
    return nil
}

func (server *Server) getRequest() *Request {
    server.reqLock.Lock()
    req := server.freeReq
    if req == nil {
        req = new(Request)
    } else {
        server.freeReq = req.next
        *req = Request{}
    }
    server.reqLock.Unlock()
    return req
}

func (server *Server) freeRequest(req *Request) {
    server.reqLock.Lock()
    req.next = server.freeReq
    server.freeReq = req
    server.reqLock.Unlock()
}

func (server *Server) getResponse() *Response {
    server.respLock.Lock()
    resp := server.freeResp
    if resp == nil {
        resp = new(Response)
    } else {
        server.freeResp = resp.next
        *resp = Response{}
    }
    server.respLock.Unlock()
    return resp
}

func (server *Server) freeResponse(resp *Response) {
    server.respLock.Lock()
    resp.next = server.freeResp
    server.freeResp = resp
    server.respLock.Unlock()
}

// 使用指定的序列化对象来读取用户的请求，返回注册的对象、方法类型、请求参数、形参值和返回值等
func (server *Server) readRequest(codec ServerCodec) (service *service, mtype *methodType, req *Request, argv, replyv reflect.Value, keepReading bool, err error) {
    service, mtype, req, keepReading, err = server.readRequestHeader(codec) // 解析请求头，获取本次请求对应的对象、方法、请求参数
    if err != nil {
        if !keepReading {
            return
        }
        codec.ReadRequestBody(nil)
        return
    }

    argIsValue := false
    if mtype.ArgType.Kind() == reflect.Ptr { // 创建请求参数
        argv = reflect.New(mtype.ArgType.Elem())
    } else {
        argv = reflect.New(mtype.ArgType)
        argIsValue = true
    }
    if err = codec.ReadRequestBody(argv.Interface()); err != nil { // 解析请求体，初始化请求参数
        return
    }
    if argIsValue {
        argv = argv.Elem()
    }

    replyv = reflect.New(mtype.ReplyType.Elem()) // 创建响应参数

    switch mtype.ReplyType.Elem().Kind() { // 如果响应参数是Map或者Slice，在此进行初始化
    case reflect.Map:
        replyv.Elem().Set(reflect.MakeMap(mtype.ReplyType.Elem()))
    case reflect.Slice:
        replyv.Elem().Set(reflect.MakeSlice(mtype.ReplyType.Elem(), 0, 0))
    }
    return
}

// 使用指定的序列化方式来读取用户请求的头部信息
func (server *Server) readRequestHeader(codec ServerCodec) (svc *service, mtype *methodType, req *Request, keepReading bool, err error) {
    req = server.getRequest()
    err = codec.ReadRequestHeader(req) // 解析请求头
    if err != nil {
        req = nil
        if err == io.EOF || err == io.ErrUnexpectedEOF {
            return
        }
        err = errors.New("rpc: server cannot decode request: " + err.Error())
        return
    }

    keepReading = true

    dot := strings.LastIndex(req.ServiceMethod, ".")
    if dot < 0 {
        err = errors.New("rpc: service/method request ill-formed: " + req.ServiceMethod)
        return
    }
    serviceName := req.ServiceMethod[:dot] // 获取对象名称和方法名称
    methodName := req.ServiceMethod[dot+1:]

    svci, ok := server.serviceMap.Load(serviceName) // 从注册的Map中查找对应的对象类型和方法类型
    if !ok {
        err = errors.New("rpc: can't find service " + req.ServiceMethod)
        return
    }
    svc = svci.(*service)
    mtype = svc.method[methodName]
    if mtype == nil {
        err = errors.New("rpc: can't find method " + req.ServiceMethod)
    }
    return
}

// 开启服务在某一个网络上的监听，然后在协程中处理Client的每一个连接请求
// 只有当监听器出错时，返回才返回
func (server *Server) Accept(lis net.Listener) {
    for {
        conn, err := lis.Accept()
        if err != nil {
            log.Print("rpc.Serve: accept:", err.Error())
            return
        }
        go server.ServeConn(conn)
    }
}

// 向默认服务中进行注册
func Register(rcvr interface{}) error { return DefaultServer.Register(rcvr) }

// 向默认服务中进行注册
func RegisterName(name string, rcvr interface{}) error {
    return DefaultServer.RegisterName(name, rcvr)
}

// 序列化接口
type ServerCodec interface {
    ReadRequestHeader(*Request) error // 解析请求头
    ReadRequestBody(interface{}) error // 解析请求体
    WriteResponse(*Response, interface{}) error // 返回响应
    Close() error // 关闭，可被调用多次，保持幂等性
}

// 开启默认RPC服务对某一个网络连接的请求
func ServeConn(conn io.ReadWriteCloser) {
    DefaultServer.ServeConn(conn)
}

// 使用指定的Codec来开启默认PRC服务
func ServeCodec(codec ServerCodec) {
    DefaultServer.ServeCodec(codec)
}

// 开启默认RPC服务的同步请求
func ServeRequest(codec ServerCodec) error {
    return DefaultServer.ServeRequest(codec)
}

// 开启默认RPC服务在网络上的监听
func Accept(lis net.Listener) { DefaultServer.Accept(lis) }

// Can connect to RPC service using HTTP CONNECT to rpcPath.
var connected = "200 Connected to Go RPC"

// 实现了HTTP处理器，用于将HTTP请求进行劫持
func (server *Server) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    if req.Method != "CONNECT" {    // 使用CONNECT HTTP方法
        w.Header().Set("Content-Type", "text/plain; charset=utf-8")
        w.WriteHeader(http.StatusMethodNotAllowed)
        io.WriteString(w, "405 must CONNECT\n")
        return
    }
    conn, _, err := w.(http.Hijacker).Hijack()
    if err != nil {
        log.Print("rpc hijacking ", req.RemoteAddr, ": ", err.Error())
        return
    }
    io.WriteString(conn, "HTTP/1.0 "+connected+"\n\n")
    server.ServeConn(conn)
}

// 将RPC和Debug的HTTP处理器注册到HTTP服务中
func (server *Server) HandleHTTP(rpcPath, debugPath string) {
    http.Handle(rpcPath, server)
    http.Handle(debugPath, debugHTTP{server})
}

// 向默认RPC服务中添加RPC和Debug的HTTP处理器
func HandleHTTP() {
    DefaultServer.HandleHTTP(DefaultRPCPath, DefaultDebugPath)
}
```

服务端的执行流程:

![RPC Server](/images/Golang-RPC服务端执行流程.jpg)

## Client实现

```Go
type ServerError string // RPC调用服务端出错信息

func (e ServerError) Error() string {
    return string(e)
}

var ErrShutdown = errors.New("connection is shut down") // 连接已关闭错误

// 代表一次RPC调用过程
type Call struct {
    ServiceMethod string      // 对象与方法名称
    Args          interface{} // 请求参数
    Reply         interface{} // 响应参数
    Error         error       // 调用出错信息
    Done          chan *Call  // 指示某次调用是否完成的chan
}

// RPC客户端，可以通过Client发起多次RPC调用
type Client struct {
    codec ClientCodec // 序列化接口

    reqMutex sync.Mutex
    request  Request // RPC请求头

    mutex    sync.Mutex
    seq      uint64
    pending  map[uint64]*Call // 所有未完成的请求，key为请求头中的Seq
    closing  bool // Client通过Close关闭请求
    shutdown bool // 服务端已经关闭
}

// 客户端序列化接口
type ClientCodec interface {
    WriteRequest(*Request, interface{}) error
    ReadResponseHeader(*Response) error
    ReadResponseBody(interface{}) error
    Close() error
}

// 发送RPC请求
func (client *Client) send(call *Call) {
    client.reqMutex.Lock()
    defer client.reqMutex.Unlock()

    client.mutex.Lock()
    if client.shutdown || client.closing {
        client.mutex.Unlock()
        call.Error = ErrShutdown
        call.done()
        return
    }
    seq := client.seq // 将Call加入到pending中
    client.seq++
    client.pending[seq] = call
    client.mutex.Unlock()

    client.request.Seq = seq
    client.request.ServiceMethod = call.ServiceMethod
    err := client.codec.WriteRequest(&client.request, call.Args) // 发送请求
    if err != nil {
        client.mutex.Lock()
        call = client.pending[seq]
        delete(client.pending, seq)
        client.mutex.Unlock()
        if call != nil {
            call.Error = err
            call.done()
        }
    }
}

// 获取服务端的返回结果
func (client *Client) input() {
    var err error
    var response Response
    for err == nil { // 循环等待服务端响应，根据Seq获取对应的Call，然后通知调用方
        response = Response{}
        err = client.codec.ReadResponseHeader(&response)
        if err != nil {
            break
        }
        seq := response.Seq
        client.mutex.Lock()
        call := client.pending[seq]
        delete(client.pending, seq)
        client.mutex.Unlock()

        switch {
        case call == nil:
            // 没有获取到请求Call，可能是WriteRequest出错，导致Call被移除
            err = client.codec.ReadResponseBody(nil)
            if err != nil {
                err = errors.New("reading error body: " + err.Error())
            }
        case response.Error != "":
            // 调用出错的情况
            call.Error = ServerError(response.Error)
            err = client.codec.ReadResponseBody(nil)
            if err != nil {
                err = errors.New("reading error body: " + err.Error())
            }
            call.done()
        default:
            // 正确返回的情况
            err = client.codec.ReadResponseBody(call.Reply)
            if err != nil {
                call.Error = errors.New("reading body " + err.Error())
            }
            call.done() // 通过调用者RPC完成
        }
    }

    client.reqMutex.Lock()
    client.mutex.Lock()
    client.shutdown = true // 服务端已关闭或出错
    closing := client.closing
    if err == io.EOF {
        if closing {
            err = ErrShutdown
        } else {
            err = io.ErrUnexpectedEOF
        }
    }
    for _, call := range client.pending {
        call.Error = err
        call.done() // 所有未完成的Call设置为出错
    }
    client.mutex.Unlock()
    client.reqMutex.Unlock()
    if debugLog && err != io.EOF && !closing {
        log.Println("rpc: client protocol error:", err)
    }
}

func (call *Call) done() {
    select {
    case call.Done <- call:
        // ok
    default:
        if debugLog {
            log.Println("rpc: discarding Call reply due to insufficient Done chan capacity")
        }
    }
}

// 创建客户端对象
func NewClient(conn io.ReadWriteCloser) *Client {
    encBuf := bufio.NewWriter(conn)
    client := &gobClientCodec{conn, gob.NewDecoder(conn), gob.NewEncoder(encBuf), encBuf}
    return NewClientWithCodec(client)
}

// 与NewClient相同，同时指定了序列化方式
func NewClientWithCodec(codec ClientCodec) *Client {
    client := &Client{
        codec:   codec,
        pending: make(map[uint64]*Call),
    }
    go client.input()
    return client
}

// gob序列化
type gobClientCodec struct {
    rwc    io.ReadWriteCloser
    dec    *gob.Decoder
    enc    *gob.Encoder
    encBuf *bufio.Writer
}

func (c *gobClientCodec) WriteRequest(r *Request, body interface{}) (err error) {
    if err = c.enc.Encode(r); err != nil {
        return
    }
    if err = c.enc.Encode(body); err != nil {
        return
    }
    return c.encBuf.Flush()
}

func (c *gobClientCodec) ReadResponseHeader(r *Response) error {
    return c.dec.Decode(r)
}

func (c *gobClientCodec) ReadResponseBody(body interface{}) error {
    return c.dec.Decode(body)
}

func (c *gobClientCodec) Close() error {
    return c.rwc.Close()
}

// 创建一个使用基于HTTP的Client
func DialHTTP(network, address string) (*Client, error) {
    return DialHTTPPath(network, address, DefaultRPCPath)
}

// 指定HTTP的PATH
func DialHTTPPath(network, address, path string) (*Client, error) {
    var err error
    conn, err := net.Dial(network, address)
    if err != nil {
        return nil, err
    }
    io.WriteString(conn, "CONNECT "+path+" HTTP/1.0\n\n") // 发送CONNECT请求

    resp, err := http.ReadResponse(bufio.NewReader(conn), &http.Request{Method: "CONNECT"})
    if err == nil && resp.Status == connected { // HTTP请求成功后，进入RPC模式
        return NewClient(conn), nil
    }
    if err == nil {
        err = errors.New("unexpected HTTP response: " + resp.Status)
    }
    conn.Close()
    return nil, &net.OpError{
        Op:   "dial-http",
        Net:  network + " " + address,
        Addr: nil,
        Err:  err,
    }
}

// 使用指定传输协议和端口创建Client
func Dial(network, address string) (*Client, error) {
    conn, err := net.Dial(network, address)
    if err != nil {
        return nil, err
    }
    return NewClient(conn), nil
}

// 关闭连接
func (client *Client) Close() error {
    client.mutex.Lock()
    if client.closing {
        client.mutex.Unlock()
        return ErrShutdown
    }
    client.closing = true
    client.mutex.Unlock()
    return client.codec.Close()
}

// 异步进行RPC调用，通过done chan来通知应用层RPC是否完成
func (client *Client) Go(serviceMethod string, args interface{}, reply interface{}, done chan *Call) *Call {
    call := new(Call)
    call.ServiceMethod = serviceMethod
    call.Args = args
    call.Reply = reply
    if done == nil {
        done = make(chan *Call, 10) // buffered.
    } else {
        // 不允许非缓冲的chan
        if cap(done) == 0 {
            log.Panic("rpc: done channel is unbuffered")
        }
    }
    call.Done = done
    client.send(call)
    return call
}

// 同步进行RPC调用，等待RPC调用完成
func (client *Client) Call(serviceMethod string, args interface{}, reply interface{}) error {
    call := <-client.Go(serviceMethod, args, reply, make(chan *Call, 1)).Done
    return call.Error
}
```

客户端的执行流程:

![RPC Client](/images/Golang-RPC客户端执行流程.jpg)