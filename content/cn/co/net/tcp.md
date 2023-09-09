---
weight: 5
title: "TCP"
---

include: [co/tcp.h](https://github.com/idealvin/coost/blob/master/include/co/tcp.h).


## tcp::Connection

`tcp::Connection` 类是对 TCP 连接的简单封装，用于实现 TCP server，客户端不需要用这个类。当服务端启用 SSL 时，tcp::Connection 会用 SSL 传输数据。



### Connection::Connection

```cpp
Connection(int sock);
Connection(void* ssl);
Connection(Connection&& c);
```

- 构造函数，`Connection` 由 `tcp::Server` 创建，用户不需要手动创建。
- 第 1 个版本构造一般的 TCP 连接，第 2 个版本构造使用 SSL 传输数据的 TCP 连接，第 3 个是移动构造函数。
- 从 v2.0.2 开始，用户不能继承 `Connection` 类。



### Connection::~Connection

```cpp
Connection::~Connection();
```

- 析构函数，调用 [close()](#connectionclose) 关闭连接。



### Connection::close

```cpp
int close(int ms = 0);
```

- 关闭连接，参数 ms > 0 时，延迟一段时间再关闭连接。
- 从 v2.0.1 开始，此方法可以在协程或非协程中调用。



### Connection::recv

```cpp
int recv(void* buf, int n, int ms=-1);
```

- 接收数据，与 [co::recv](../sock/#corecv) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回值 >0，超时或发生错误时返回值 <0，对端关闭连接时返回 0。



### Connection::recvn

```cpp
int recvn(void* buf, int n, int ms=-1);
```

- 接收指定长度的数据，与 [co::recvn](../sock/#corecvn) 类似。
- 此方法成功时返回 n，超时或发生错误时返回值 <0，对端关闭连接时返回 0。



### Connection::reset

```cpp
int reset(int ms = 0)
```

- 重置 TCP 连接，与 [close()](#connectionclose) 不同，它不会进入 `TIME_WAIT` 状态。参数 ms > 0 时，延迟一段时间再重置连接。
- 此方法必须在 I/O 线程(一般是进行 I/O 操作的协程)中调用。



### Connection::send

```cpp
int send(const void* buf, int n, int ms=-1);
```

- 发送数据，与 [co::send()](../sock/#cosend) 类似。
- 此方法成功时返回 n，超时或发生错误时返回值 <=0。



### Connection::socket

```cpp
int socket() const;
```

- 返回内部的 socket 描述符，连接已关闭时返回 -1。



### Connection::strerror

```cpp
const char* strerror() const;
```

- Connection 中的方法报错时，可以调用此方法查看错误信息。




## tcp::Server

`tcp::Server` 是基于协程的 TCP 服务端，它的特性如下：

- 支持 IPv4 与 IPv6。
- 支持 SSL (需要 openssl)。
- 采用一个连接一个协程的模型。


### Server::Server

```cpp
Server();
```

- 构造函数，初始化。



### Server::conn_num

```cpp
uint32 conn_num() const;
```

- 返回当前的客户端连接数。



### Server::on_connection

```cpp
Server& on_connection(std::function<void(Connection)>&& f);
Server& on_connection(const std::function<void(Connection)>& f);

template<typename T>
Server& on_connection(void (T::*f)(Connection), T* o);
```

- 设置处理客户端连接的回调函数。
- 第 1, 2 个版本中，参数 f 是 `void f(Connection)` 类型的函数，或 `std::function<void(Connection)>` 类型的函数对象。
- 第 3 个版本中，参数 f 是类中的方法，参数 o 是 T 类型的指针。
- 从 v2.0.2 开始，f 的参数改为 tcp::Connection 对象，而非指针，用户不需要手动 delete。
- 服务端接收到新的客户端连接时，会新建一个协程，并在协程中调用此方法设置的回调函数，处理新连接上的数据。


- 示例

```cpp
void f(tcp::Connection conn);

tcp::Server s;
s.on_connection(f);

void f(tcp::Connection conn) {
    while (true) {
        conn.recv(...);
        process(...);
        conn.send(...);
    }
    
    conn.close();
}
```



### Server::on_exit

```cpp
Server& on_exit(std::function<void()>&& cb);
```

- 设置一个 callback，它将在 server 退出时被调用。



### Server::start

```cpp
void start(const char* ip, int port, const char* key=0, const char* ca=0);
```

- 启动 TCP server，此方法不会阻塞当前线程。
- 参数 ip 是服务器 ip，可以是 IPv4 或 IPv6 地址，参数 port 是服务器端口。
- 参数 key 是存放 SSL private key 的 PEM 文件路径，参数 ca 是存放 SSL 证书的 PEM 文件路径，默认 key 和 ca 是 NULL，不启用 SSL。
- 从 v3.0 开始，server 启动后就不再依赖于 tcp::Server 对象。


- 示例

```cpp
void f(tcp::Connection conn);
tcp::Server().on_connection(f).start("0.0.0.0", 7788);
```



### Server::exit

```cpp
void exit();
```

- v2.0.2 新增。
- 退出 TCP server，关闭 listening socket，不再接收新的连接。
- **此方法不会关闭之前已经建立的连接**。
- 若需要在 server 退出后，关闭之前建立的连接，可以参考 [test/tcp2.cc](https://github.com/idealvin/cocoyaxi/blob/master/test/so/tcp2.cc) 或 co 中 `http::Server` 与 `rpc::Server` 的实现。




## tcp::Client

`tcp::Client` 是基于协程的 TCP 客户端，它有如下特性：

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

- 构造函数。参数 ip 是服务端 ip，可以是域名、IPv4 或 IPv6 地址；参数 port 是服务端口；参数 use_ssl 表示是否启用 SSL 传输，默认为 false。
- 第 2 个版本是拷贝构造函数，仅拷贝 ip, port, use_ssl。
- tcp::Client 构建时，并没有建立连接。
- 一般建议在调用 recv, send 前，判断连接是否建立，没有的话就调用 [connect()](#clientconnect) 方法建立连接，这种方式可以实现自动重连。



### Client::~Client

```cpp
Client::~Client();
```

- 析构函数，调用 [disconnect()](#clientdisconnect) 方法关闭连接。



### Client::close

```cpp
void close();
```

- 关闭连接，与 disconnect() 相同。



### Client::connect

```cpp
bool connect(int ms);
```

- 建立连接，参数 ms 是超时时间，单位为毫秒。
- 此方法必须在协程中调用。
- 此方法成功时返回 true，否则返回 false。失败时，用户可以调用 [strerror()](#clientstrerror) 方法查看错误信息。



### Client::connected

```cpp
bool connected() const;
```

- 判断是否已经建立连接。



### Client::disconnect

```cpp
void disconnect();
```

- 从 v2.0.1 开始，可以在协程或非协程中调用。
- 多次调用此方法是安全的，析构函数中会自动调用此方法。



### Client::recv

```cpp
int recv(void* buf, int n, int ms=-1);
```

- 接收数据，与 [co::recv()](../sock/#corecv) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回值 >0，超时或发生错误时返回值 <0，对端关闭连接时返回 0。



### Client::recvn

```cpp
int recvn(void* buf, int n, int ms=-1);
```

- 接收指定长度的数据，与 [co::recvn()](../sock/#corecvn) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回 n，超时或发生错误时返回值 <0，对端关闭连接时返回 0。



### Client::send

```cpp
int send(const void* buf, int n, int ms=-1);
```

- 发送数据，与 [co::send()](../sock/#cosend) 类似。
- 此方法必须在协程中调用。
- 此方法成功时返回 n，超时或发生错误时返回值 <=0。



### Client::socket

```cpp
int socket() const;
```

- 返回内部的 socket 描述符。
- 未建立连接或连接已经关闭时，返回值是 -1。



### Client::strerror

```cpp
const char* strerror() const;
```

- tcp::Client 中的方法报错时，可以调用此方法查看错误信息。




## TCP 服务端代码示例

```cpp
void on_connection(tcp::Connection conn) {
    char buf[8] = { 0 };

    while (true) {
        int r = conn.recv(buf, 8);
        if (r == 0) {         /* client close the connection */
            conn.close();
            break;
        } else if (r < 0) { /* error */
            conn.reset(3000);
            break;
        } else {
            LOG << "server recv " << fastring(buf, r);
            LOG << "server send pong";
            r = conn.send("pong", 4);
            if (r <= 0) {
                LOG << "server send error: " << conn.strerror();
                conn.reset(3000);
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

co::pool pool(
    []() {return (void*) new tcp::Client(*proto); },
    [](void* p) {delete (tcp::Client*) p;}
);

void client_fun() {
    co::pool_guard<tcp::Client> c(pool);
    
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

- 上面的例子中，我们用 [co::pool](../../concurrency/coroutine/pool/) 缓存客户端连接，不同协程可以共用 pool 中的连接。
