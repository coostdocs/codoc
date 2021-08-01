---
weight: 14
title: "TCP"
---


include: [co/so/tcp.h](https://github.com/idealvin/co/blob/master/include/co/so/tcp.h).


## tcp::Connection


`tcp::Connection` 类是对 TCP 连接的简单封装，用于实现服务端代码，客户端不需要用这个类。当服务端启用 SSL 时，tcp::Connection 会用 SSL 传输数据。




### Connection::Connection
```cpp
Connection(sock_t s);
```

- 构造函数，参数 s 是 socket 描述符。





### Connection::~Connection
```cpp
Connection::~Connection();
```

- 析构函数，调用 close() 关闭连接。





### Connection::close
```cpp
virtual int close(int ms = 0);
```

- 关闭连接，参数 ms > 0 时，延迟一段时间再关闭连接。
- 此方法必须在 I/O 线程(一般是进行 I/O 操作的协程)中调用。





### Connection::recv
```cpp
virtual int recv(void* buf, int n, int ms=-1);
```

- 接收数据，与 [co::recv](../../coroutine/#corecv) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回值 >0，超时或发生错误时返回值 <0，对端关闭连接时返回 0。





### Connection::recvn
```cpp
virtual int recvn(void* buf, int n, int ms=-1);
```

- 接收指定长度的数据，与 [co::recvn](../../coroutine/#corecvn) 类似。
- 此方法成功时返回 n，超时或发生错误时返回值 <0，对端关闭连接时返回 0。





### Connection::reset
```cpp
virtual int reset(int ms = 0)
```

- 重置 TCP 连接，与 close() 不同，它不会进入 `TIME_WAIT` 状态。参数 ms > 0 时，延迟一段时间再重置连接。
- 此方法必须在 I/O 线程(一般是进行 I/O 操作的协程)中调用。





### Connection::send
```cpp
virtual int send(const void* buf, int n, int ms=-1);
```

- 发送数据，与 [co::send()](../../coroutine/#cosend) 类似。
- 此方法成功时返回 n，超时或发生错误时返回值 <=0。





### Connection::socket
```cpp
sock_t socket() const;
```

- 返回内部的 socket 描述符，连接已关闭时返回 -1。





### Connection::strerror
```cpp
virtual const char* strerror() const;
```

- Connection 中的方法出错时，可以调用此方法查看错误信息。







## tcp::Server


`tcp::Server` 是基于协程的 TCP 服务端，它的特性如下：

- 支持 IPv4 与 IPv6。
- 支持 SSL (需要 openssl)。
- 采用一个连接一个协程的模型。





### Server::Server
```cpp
Server() = default;
```

- 构造函数，什么也不做。





### Server::on_connection
```cpp
void on_connection(std::function<void(Connection*)>&& f);
template<typename T> void on_connection(void (T::*f)(Connection*), T* o);
```

- 设置处理客户端连接的回调函数。
- 第 1 个版本中，参数 f 是 `void f(Connection*)` 类型的函数，或 `std::function<void(Connection*)>` 类型的函数对象。
- 第 2 个版本中，参数 f 是类中的方法，参数 o 是 T 类型的指针。
- f 的参数是一个 tcp::Connection 类型的指针，它是用 operator new 创建的，用户在关闭连接后需要 delete 掉该指针。
- 服务端接收到新的客户端连接时，会新建一个协程，并在协程中调用此方法设置的回调函数，处理新连接上的数据。



- 示例
```cpp
void f(tcp::Connection* conn);

tcp::Server s;
s.on_connection(f);

void f(tcp::Connection* conn) {
    std::unique_ptr<tcp::Connection> c(conn);
    
    while (true) {
        conn->recv(...);
        process(...);
        conn->send(...);
    }
    
    conn->close();
}
```




### Server::start
```cpp
virtual void start(const char* ip, int port, const char* key=NULL, const char* ca=NULL);
```

- 启动 TCP server，此方法不会阻塞当前线程。
- 参数 ip 是服务器 ip，可以是 IPv4 或 IPv6 地址，参数 port 是服务器端口。
- 参数 key 是存放 SSL private key 的 PEM 文件路径，参数 ca 是存放 SSL 证书的 PEM 文件路径，默认 key 和 ca 是 NULL，不启用 SSL。







## tcp::Client


`tcp::Client` 是基于协程的 TCP 客户端，它的特性如下：

- 支持 IPv4 与 IPv6。
- 支持 SSL (需要安装 openssl)。
- 一个客户端对象，对应一个连接。
- 它必须在协程中使用。
- 它不是协程安全的，同一时刻，不能有多个协程对它进行操作。





### Client::Client
```cpp
Client(const char* ip, int port, bool use_ssl=false);
Client(const Client& c);
```

- 构造函数。参数 ip 是服务器的 ip，可以是域名、IPv4 或 IPv6 地址；参数 port 是服务器端口；参数 use_ssl 表示是否启用 SSL 传输，默认为 false，不启用 SSL。
- 第 2 个版本是拷贝构造函数，仅拷贝 ip, port, use_ssl。
- tcp::Client 构建时，并没有建立连接。
- 一般建议在调用 recv, send 前，判断连接是否建立，没有的话就调用 connect 方法建立连接，这种方式可以实现自动重连。





### Client::~Client
```cpp
Client::~Client();
```

- 析构函数，调用 disconnect() 方法关闭连接。
- 由于 disconnect() 必须在 I/O 线程中调用，用户一般需要在进行 I/O 操作的协程中析构 tcp::Client 对象。





### Client::close
```cpp
void close();
```

- 关闭连接，与 disconnect() 相同。





### Client::connect
```cpp
virtual bool connect(int ms);
```

- 建立连接，参数 ms 是超时时间，单位为毫秒。
- 此方法必须在协程中调用。
- 此方法成功时返回 true，否则返回 false。失败时，用户可以用 `strerror()` 方法查看错误信息。





### Client::connected
```cpp
virtual bool connected() const;
```

- 判断是否已经建立连接。





### Client::disconnect
```cpp
virtual void disconnect();
```

- 关闭连接，必须在 I/O 线程(一般是进行 I/O 操作的协程)中调用。
- 多次调用此方法是安全的，析构函数中会自动调用此方法。





### Client::recv
```cpp
virtual int recv(void* buf, int n, int ms=-1);
```

- 接收数据，与 [co::recv()](../../coroutine/#corecv) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回值 >0，超时或发生错误时返回值 <0，对端关闭连接时返回 0。





### Client::recvn
```cpp
virtual int recvn(void* buf, int n, int ms=-1);
```

- 接收指定长度的数据，与 [co::recvn()](../../coroutine/#corecvn) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回 n，超时或发生错误时返回值 <0，对端关闭连接时返回 0。





### Client::send
```cpp
virtual int send(const void* buf, int n, int ms=-1);
```

- 发送数据，与 [co::send()](../../coroutine/#cosend) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回 n，超时或发生错误时返回值 <=0。





### Client::socket
```cpp
sock_t socket() const;
```

- 返回内部的 socket 描述符。
- 未建立连接或连接已经关闭时，返回值是 -1。





### Client::strerror
```cpp
virtual const char* strerror() const;
```

- tcp::Client 中的方法出错时，可以调用此方法查看错误信息。







## TCP 服务端代码示例


```cpp
void on_connection(tcp::Connection* conn) {
    std::unique_ptr<tcp::Connection> c(conn);
    char buf[8] = { 0 };

    while (true) {
        int r = conn->recv(buf, 8);
        if (r == 0) {         /* client close the connection */
            conn->close();
            break;
        } else if (r < 0) { /* error */
            conn->reset(3000);
            break;
        } else {
            LOG << "server recv " << fastring(buf, r);
            LOG << "server send pong";
            r = conn->send("pong", 4);
            if (r <= 0) {
                LOG << "server send error: " << conn->strerror();
                conn->reset(3000);
                break;
            }
        }
    }
}

tcp::Server s;
s.on_connection(on_connection);
s.start("0.0.0.0", 7788);                                    // no ssl
s.start("0.0.0.0", 7788, "privkey.pem", "certificate.pem");  // use ssl
```

- 上面的例子实现了一个简单的 ping-pong server，收到客户端发送的 ping 时，回复一个 pong。







## TCP 客户端代码示例


```cpp
bool use_ssl = false;
std::unique_ptr<tcp::Client> proto;

co::Pool pool(
    []() {return (void*) new tcp::Client(*proto); },
    [](void* p) {delete (tcp::Client*) p;}
);

void client_fun() {
    co::PoolGuard<tcp::Client> c(pool);
    
    if (!c->connect(3000)) {
        LOG << "connect failed: "<< c->strerror();
        return;
    }

    char buf[8] = {0 };

    while (true) {
        LOG << "client send ping";
        int r = c->send("ping", 4);
        if (r <= 0) {
            LOG << "client send error: "<< c->strerror();
            break;
        }

        r = c->recv(buf, 8);
        if (r < 0) {
            LOG << "client recv error: "<< c->strerror();
            break;
        } else if (r == 0) {
            LOG << "server close the connection";
            break;
        } else {
            LOG << "client recv "<< fastring(buf, r) <<'\n';
            co::sleep(3000);
        }
    }
}

proto.reset(new tcp::Client("127.0.0.1", 7788, use_ssl));
for (int i = 0; i <8; ++i) {
    go(client_fun);
}
```

- 上面的例子中，我们用 co::Pool 缓存客户端连接，不同协程可以共用 co::Pool 中的连接。
- co::PoolGuard 构建时自动从 co::Pool 中拉取一个空闲连接，析构时自动将该连接放回 co::Pool 中。
