---
weight: 2
title: "Socket Programming"
---

## Header

```cpp
#include "co/sock.h"
```

The API is in the `co` namespace. Before use, you need to call `flag::parse(argc, argv);` at the beginning of `main`.

`sock_t` is in the global namespace.

## Overview

- Coroutine-based socket API.
- I/O-related APIs do not block the scheduling thread and must be called inside a coroutine.
- On error, use `co::error()` / `co::strerror()` to get error information.

## Address co::sockaddr

```cpp
struct sockaddr {
    sockaddr();
    sockaddr(const char* host, uint16 port);
    bool valid() const;
    int af() const;
};
```

- Supports IPv4, IPv6, and domain name resolution.
- `af()` returns `AF_INET` or `AF_INET6`.
- Can be printed directly, in the format `127.0.0.1:8080`.

## I/O Event Types

```cpp
enum ev_t {
    ev_read = 1,
    ev_write = 2,
};
```

- Bitwise OR combination is not supported.

## Creating a Socket

```cpp
sock_t co::tcp_socket(int af=0);
sock_t co::udp_socket(int af=0);
sock_t co::tcp_server_socket(const char* host, uint16 port, int backlog=4096);
```

- `af=0` defaults to `AF_INET`.
- The created socket is already non-blocking.
- `tcp_server_socket`: creates a TCP socket, binds the address, and listens.

## Socket Operations

```cpp
int co::close(sock_t fd, int ms=0);
int co::bind(sock_t fd, const sockaddr& addr);
int co::listen(sock_t fd, int backlog=4096);
int co::connect(sock_t fd, const sockaddr& addr, int ms=-1);
sock_t co::accept(sock_t fd, sockaddr* addr=nullptr);
```

- `close(fd, ms)`: delays closing by `ms` milliseconds to prevent malicious attacks.
- `connect`: `ms=-1` means no timeout; the timeout error code is `ETIMEDOUT`.
- `accept` returns the socket on success and `(sock_t)-1` on error; the others return 0 on success and -1 on error.

## Send and Receive

```cpp
int co::recv(sock_t fd, void* buf, int n, int ms=-1);
int co::recvn(sock_t fd, void* buf, int n, int ms=-1);
int co::recvfrom(sock_t fd, void* buf, int n, sockaddr& addr, int ms=-1);
int co::send(sock_t fd, const void* buf, int n, int ms=-1);
int co::sendto(sock_t fd, const void* buf, int n, const sockaddr& addr, int ms=-1);
```

- `recv`: single read, may return fewer than `n` bytes.
- `recvn`: reads exactly `n` bytes, suitable for protocols with a fixed-length header.
- `recvfrom`: UDP receive; when the function returns, `addr` is filled with the sender's address.
- `send`: returning `n` on success means all data was sent; the state after an error or timeout is undefined.
- `sendto`: UDP send.
- `ms=-1` means no timeout; the timeout error code is `ETIMEDOUT`.

## Socket Options

```cpp
void co::set_reuseaddr(sock_t fd);
void co::set_tcp_nodelay(sock_t fd);
void co::set_tcp_keepalive(sock_t fd);
void co::set_send_buffer_size(sock_t fd, int n);
void co::set_recv_buffer_size(sock_t fd, int n);
void co::set_nonblock(sock_t fd);
void co::set_cloexec(sock_t fd); // non-Windows
```

- `set_send_buffer_size` / `set_recv_buffer_size` must be called before connecting.

## Others

```cpp
int co::reset_tcp_socket(sock_t fd, int ms=0);
co::sockaddr co::peeraddr(sock_t fd);
```

- `reset_tcp_socket`: resets the connection without entering `timed_wait`; generally used on the server side.
- `peeraddr`: returns the peer address.

## Examples

### TCP Client

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

### TCP Server

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
