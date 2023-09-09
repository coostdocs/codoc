---
weight: 2
title: "socket 编程"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## Socket APIs

co 提供了常用的协程化的 socket API，以支持基于协程的网络编程。

**大部分 API 形式上与原生的 socket API 保持一致**，这样可以减轻用户的学习负担，熟悉 socket 编程的用户可以轻松上手。

这些 API 大部分需要在协程中使用，它们在 I/O 阻塞或调用 sleep 等操作时，调度线程会挂起当前协程，切换到其他等待中的协程运行，调度线程本身并不会阻塞。借助这些 API，用户可以**轻松的实现高并发、高性能的网络程序**。



### 术语约定

**阻塞**

在描述 co 中的一些 socket API 时，会用到阻塞一词，如 accept, recv，文档中说它们会阻塞，是指当前的协程会阻塞，而当前的调度线程并不会阻塞(可以切换到其他协程运行)。用户看到的是协程，而不是调度线程，因此从用户的角度看，它们是阻塞的。实际上，这些 API 内部使用 **non-blocking socket**，并不会真的阻塞，只是在 socket 上没有数据可读或者无法立即写入数据时，调度线程会挂起当前协程，当 socket 变为可读或可写时，调度线程会重新唤起该协程，继续 I/O 操作。


**non-blocking socket**

**co 中的 socket API 必须使用 non-blocking socket**，在 windows 平台还要求 socket 支持 [overlapped I/O](https://support.microsoft.com/en-us/help/181611/socket-overlapped-i-o-versus-blocking-nonblocking-mode)，win32 API 创建的 socket 默认都支持 overlapped I/O，用户一般不需要担心这个问题。为了叙述方便，这里约定文档中说到 non-blocking socket 时，同时也表示它在 windows 上支持 overlapped I/O。



### co::socket

```cpp
1. sock_t socket(int domain, int type, int proto);
2. sock_t tcp_socket(int domain=AF_INET);
3. sock_t udp_socket(int domain=AF_INET);
```

- 创建 socket。
- 1, 形式上与原生 API 完全一样，在 linux 系统可以用 `man socket` 查看参数详情。
- 2, 创建一个 TCP socket。
- 3, 创建一个 UDP socket。
- 参数 `domain` 一般是 AF_INET 或 AF_INET6，前者表示 ipv4，后者表示 ipv6。
- **这些函数返回一个 non-blocking socket**。发生错误时，返回值是 -1，可以调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::accept

```cpp
sock_t accept(sock_t fd, void* addr, int* addrlen);
```

- 在指定 socket 上接收客户端连接，参数 fd 是之前调用 [listen()](#colisten) 监听的 non-blocking socket，参数 addr 与 addrlen 用于接收客户端的地址信息，`*addrlen` 的初始值是 addr 所指向 buffer 的长度。如果用户不需要客户端地址信息，可以将 addr 与 addrlen 设置为 NULL。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到有新的连接进来，或者发生错误。
- 此函数成功时**返回一个 non-blocking socket**，发生错误时返回 -1，可以调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::bind

```cpp
int bind(sock_t fd, const void* addr, int addrlen);
```

- 给 socket 绑定 ip 地址，参数 addr 与 addrlen 是地址信息，与原生 API 相同。
- 此函数成功时返回 0，否则返回 -1，可以调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::close

```cpp
int close(sock_t fd, int ms=0);
```

- 关闭 socket。
- 在 2.0.0 及之前的版本中，此函数必须在进行 I/O 操作的线程中调用。从 2.0.1 版本开始，此函数可以在协程或非协程中调用。
- 参数 ms 大于 0 时，会先调用 `co::sleep(ms)` 将当前协程挂起一段时间，再关闭 socket。一般只在 server 端将 ms 设置为大于 0 的值，可以在一定程度上缓解非法的网络攻击。
- 此函数内部已经处理了 `EINTR` 信号，用户无需考虑。
- 此函数成功时返回 0，否则返回 -1，可以调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::connect

```cpp
int connect(sock_t fd, const void* addr, int addrlen, int ms=-1);
```

- 在指定 socket 上创建到指定地址的连接，参数 fd 必须是 non-blocking 的，参数 addr 与 addrlen 是地址信息，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到连接完成，或者超时、发生错误。
- 此函数成功时返回 0，超时或发生错误返回 -1，用户可以调用 [co::timeout()](../../concurrency/coroutine/api/#cotimeout) 判断是否超时，调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::listen

```cpp
int listen(sock_t fd, int backlog=1024);
```

- 监听指定的 socket，参数 fd 是已经调用 [bind()](#cobind) 绑定 ip 及端口的 socket。
- 此函数成功时返回 0，否则返回 -1，可以调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::recv

```cpp
int recv(sock_t fd, void* buf, int n, int ms=-1);
```

- 在指定 socket 上接收数据，参数 fd 必须是 non-blocking 的，参数 buf 是用于接收数据的 buffer，参数 n 是 buffer 长度，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 在 Windows 平台，此函数只适用于 TCP 等 stream 类型的 socket。
- 此函数会阻塞，直到有数据进来，或者超时、发生错误。
- 此函数成功时返回接收的数据长度(可能小于 n)，对端关闭连接时返回 0，超时或发生错误返回 -1，用户可以调用 [co::timeout()](../../concurrency/coroutine/api/#cotimeout) 判断是否超时，调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::recvn

```cpp
int recvn(sock_t fd, void* buf, int n, int ms=-1);
```

- 在指定 socket 上接收指定长度的数据，参数 fd 必须是 non-blocking 的，参数 buf 是用于接收数据的 buffer，参数 n 是要接收数据的长度，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到 n 字节的数据全部接收完，或者超时、发生错误。
- 此函数成功时返回 n，对端关闭连接时返回 0，超时或发生错误返回 -1，用户可以调用 [co::timeout()](../../concurrency/coroutine/api/#cotimeout) 判断是否超时，调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::recvfrom

```cpp
int recvfrom(sock_t fd, void* buf, int n, void* src_addr, int* addrlen, int ms=-1);
```

- 与 [co::recv](#corecv) 类似，只是可以用参数 src_addr 与 addrlen 接收源地址信息，`*addrlen` 的初始值是 src_addr 所指向 buffer 的长度，如果用户不需要源地址信息，可以将 addr 与 addrlen 设置为 NULL。
- 一般**建议只用此函数接收 UDP 数据**。



### co::send

```cpp
int send(sock_t fd, const void* buf, int n, int ms=-1);
```

- 向指定 socket 上发送数据，参数 fd 必须是 non-blocking 的，参数 buf 与 n 是要发送的数据及长度，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 在 Windows 平台，此函数只适用于 TCP 等 stream 类型的 socket。
- 此函数会阻塞，直到 n 字节的数据全部发送完，或者超时、发生错误。
- 此函数成功时返回 n，超时或发生错误返回 -1，用户可以调用 [co::timeout()](../../concurrency/coroutine/api/#cotimeout) 判断是否超时，调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### co::sendto

```cpp
int sendto(sock_t fd, const void* buf, int n, const void* dst_addr, int addrlen, int ms=-1);
```

- 向指定的地址发送数据，当 dst_addr 为 NULL，addrlen 为 0 时，与 [co::send](#cosend) 等价。
- 一般**建议只用此函数发送 UDP 数据**。
- fd 是 UDP socket 时，n 最大是 65507。



### co::shutdown

```cpp
int shutdown(sock_t fd, char c='b');
```

- 此函数一般用于半关闭 socket，参数 c 为 `'r'` 时表示关闭读，为 `'w'` 时表示关闭写，默认为 `'b'`，关闭读与写。
- 一般建议在进行 IO 操作的线程中调用此函数。
- 此函数成功时返回 0，否则返回 -1，可以调用 [co::error()](../../other/error/#coerror) 获取错误信息。



### ———————————
### co::getsockopt

```cpp
int getsockopt(sock_t fd, int lv, int opt, void* optval, int* optlen);
```

- 获取 socket option 信息，与原生 API 完全一样，man getsockopt 看详情。



### co::setsockopt

```cpp
int setsockopt(sock_t fd, int lv, int opt, const void* optval, int optlen);
```

- 设置 socket option 信息，与原生 API 完全一样，man setsockopt 看详情。



### co::set_nonblock

```cpp
void set_nonblock(sock_t fd);
```

- 给 socket 设置 O_NONBLOCK 选项。



### co::set_reuseaddr

```cpp
void set_reuseaddr(sock_t fd);
```

- 给 socket 设置 SO_REUSEADDR 选项，一般 server 端的 listening socket 需要设置这个选项，防止 server 重启后 bind 失败。



### co::set_recv_buffer_size

```cpp
void set_recv_buffer_size(sock_t fd, int n);
```

- 设置 socket 的接收缓冲区大小，必须在 socket 连接前调用此函数。



### co::set_send_buffer_size

```cpp
void set_send_buffer_size(sock_t fd, int n);
```

- 设置 socket 的发送缓冲区大小，必须在 socket 连接前调用此函数。



### co::set_tcp_keepalive

```cpp
void set_tcp_keepalive(sock_t fd);
```

- 给 socket 设置 SO_KEEPALIVE 选项。



### co::set_tcp_nodelay

```cpp
void set_tcp_nodelay(sock_t fd);
```

- 给 socket 设置 TCP_NODELAY 选项。



### co::reset_tcp_socket

```cpp
int reset_tcp_socket(sock_t fd, int ms=0);
```

- 重置 TCP 连接，与 [co::close](#coclose) 类似，但主动调用方不会进入 TIME_WAIT 状态。
- 一般只有 server 端会调用此函数，用于主动关闭客户端连接，同时避免进入 TIME_WAIT 状态。




### ———————————
### co::addr2str

```cpp
1. fastring addr2str(const struct sockaddr_in* addr);
2. fastring addr2str(const struct sockaddr_in6* addr);
3. fastring addr2str(const void* addr, int len);
```

- 将 sockaddr 地址转换成 `"ip:port"` 形式的字符串。
- 1 用于 ipv4 地址，2 用于 ipv6 地址，3 根据 `len` 选择调用版本 1 或版本 2。

- 示例

```cpp
struct sockaddr_in addr;
co::init_addr(&addr, "127.0.0.1", 80);
co::addr2str(&addr);                // "127.0.0.1:80"
co::addr2str(&addr, sizeof(addr));  // "127.0.0.1:80"
```



### co::init_addr

```cpp
1. bool init_addr(struct sockaddr_in* addr, const char* ip, int port);
2. bool init_addr(struct sockaddr_in6* addr, const char* ip, int port);
```

- 用 ip 及 port 初始化 sockaddr 结构。
- 1 用于 ipv4 地址，2 用于 ipv6 地址。

- 示例

```cpp
union {
    struct sockaddr_in  v4;
    struct sockaddr_in6 v6;
} addr;

co::init_addr(&addr.v4, "127.0.0.1", 7777);
co::init_addr(&addr.v6, "::", 7777);
```



### co::init_ip_addr

```cpp
bool init_ip_addr(struct sockaddr_in* addr, const char* ip, int port);
bool init_ip_addr(struct sockaddr_in6* addr, const char* ip, int port);
```

- v3.0.1 标记为 **deprecated**，建议用 [co::init_addr](#coinit_addr) 取代之。



### co::peer

```cpp
fastring peer(sock_t fd);
```

- 获取 peer 端的地址信息，返回值是 `"ip:port"` 形式的字符串。
