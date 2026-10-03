---
weight: 2
title: "socket 编程"
---


## 头文件

```cpp
#include "co/sock.h"
```

API 在 `co` 命名空间。使用前需在 `main` 开头调用 `flag::parse(argc, argv);`。

`sock_t` 在全局命名空间。

## 概述

- 基于协程的 socket API。
- I/O 相关 API 不阻塞调度线程，必须在协程中调用。
- 出错可用 `co::error()` / `co::strerror()` 获取错误信息。


## 地址 co::sockaddr

```cpp
struct sockaddr {
    sockaddr();
    sockaddr(const char* host, uint16 port);
    bool valid() const;
    int af() const;
};
```

- 支持 IPv4、IPv6、域名解析。
- `af()` 返回 `AF_INET` 或 `AF_INET6`。
- 可直接打印，格式 `127.0.0.1:8080`。


## I/O 事件类型

```cpp
enum ev_t {
    ev_read = 1,
    ev_write = 2,
};
```

- 不支持按位或组合。


## 创建 socket

```cpp
sock_t co::tcp_socket(int af=0);
sock_t co::udp_socket(int af=0);
sock_t co::tcp_server_socket(const char* host, uint16 port, int backlog=4096);
```

- `af=0` 默认 `AF_INET`。
- 创建的 socket 已是 non-block。
- `tcp_server_socket`：创建 TCP socket，绑定地址并监听。


## socket 操作

```cpp
int co::close(sock_t fd, int ms=0);
int co::bind(sock_t fd, const sockaddr& addr);
int co::listen(sock_t fd, int backlog=4096);
int co::connect(sock_t fd, const sockaddr& addr, int ms=-1);
sock_t co::accept(sock_t fd, sockaddr* addr=nullptr);
```

- `close(fd, ms)`：延迟 `ms` 毫秒关闭，防恶意攻击。
- `connect`：`ms=-1` 不超时，超时错误码 `ETIMEDOUT`。
- `accept` 成功返回 socket，错误返回 `(sock_t)-1`，其余成功返回 0，错误返回 -1。


## 收发

```cpp
int co::recv(sock_t fd, void* buf, int n, int ms=-1);
int co::recvn(sock_t fd, void* buf, int n, int ms=-1);
int co::recvfrom(sock_t fd, void* buf, int n, sockaddr& addr, int ms=-1);
int co::send(sock_t fd, const void* buf, int n, int ms=-1);
int co::sendto(sock_t fd, const void* buf, int n, const sockaddr& addr, int ms=-1);
```

- `recv`：单次读取，可能少于 `n`。
- `recvn`：精确读取 `n` 字节，适合固定长度头部协议。
- `recvfrom`：UDP 接收；函数返回时，`addr` 中填充发送方地址。
- `send`：成功返回 `n` 表示全部发送；出错或超时状态未定义。
- `sendto`：UDP 发送。
- `ms=-1` 不超时，超时错误码 `ETIMEDOUT`。


## socket 选项

```cpp
void co::set_reuseaddr(sock_t fd);
void co::set_tcp_nodelay(sock_t fd);
void co::set_tcp_keepalive(sock_t fd);
void co::set_send_buffer_size(sock_t fd, int n);
void co::set_recv_buffer_size(sock_t fd, int n);
void co::set_nonblock(sock_t fd);
void co::set_cloexec(sock_t fd); // 非 Windows
```

- `set_send_buffer_size` / `set_recv_buffer_size` 必须在连接前调用。


## 其它

```cpp
int co::reset_tcp_socket(sock_t fd, int ms=0);
co::sockaddr co::peeraddr(sock_t fd);
```

- `reset_tcp_socket`：重置连接，不进 `timed_wait`，一般用于服务端。
- `peeraddr`：返回对端地址。


## 示例

### TCP 客户端

```cpp
#include "co/co.h"
#include "co/sock.h"
#include "co/print.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    go([] {
        co::sockaddr addr("127.0.0.1", 8080);
        sock_t fd = co::tcp_socket(addr.af());

        int r = co::connect(fd, addr, 3000);
        if (r != 0) {
            co::println("connect failed: ", co::strerror());
            co::close(fd);
            return;
        }

        const char* msg = "hello";
        r = co::send(fd, msg, 5, 3000);
        co::println("sent ", r, " bytes");

        char buf[128];
        r = co::recv(fd, buf, sizeof(buf), 3000);
        if (r > 0) co::println("recv: ", co::string(buf, r));

        co::close(fd);
    });

    co::sleep(5000);
    return 0;
}
```

### TCP 服务端

```cpp
#include "co/co.h"
#include "co/sock.h"
#include "co/print.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    go([] {
        sock_t s = co::tcp_server_socket("0.0.0.0", 8080);
        if (s == (sock_t)-1) {
            co::println("listen failed: ", co::strerror());
            return;
        }

        while (true) {
            co::sockaddr addr;
            sock_t fd = co::accept(s, &addr);
            if (fd == (sock_t)-1) break;

            go([fd, addr] {
                co::println("accepted: ", addr);

                char buf[1024];
                int n = co::recvn(fd, buf, 4);
                if (n == 4) {
                    uint32 body_len = *(uint32*)buf;
                    co::string body;
                    body.resize(body_len);
                    co::recvn(fd, body.data(), body_len);
                    co::println("body: ", body);
                }

                co::close(fd);
            });
        }

        co::close(s);
    });

    co::sleep(60000);
    return 0;
}
```

### UDP

```cpp
#include "co/co.h"
#include "co/sock.h"
#include "co/print.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    go([] {
        sock_t fd = co::udp_socket();
        co::sockaddr addr("127.0.0.1", 8080);

        const char* msg = "ping";
        co::sendto(fd, msg, 4, addr, 3000);

        char buf[128];
        co::sockaddr peer;
        int n = co::recvfrom(fd, buf, sizeof(buf), peer, 3000);
        if (n > 0) co::print("recvfrom ", peer, ": ", co::string(buf, n), '\n');

        co::close(fd);
    });

    co::sleep(5000);
    return 0;
}
```
