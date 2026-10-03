---
weight: 3
title: "TCP"
---

## Header

```cpp
#include "co/tcp.h"
```

The API is in the `co` namespace, and aliases are also provided:

```cpp
namespace tcp {
using client = co::tcp_client;
using server = co::tcp_server;
}
```

Before use, you need to call `flag::parse(argc, argv);` at the beginning of `main`.

## tcp_client

```cpp
struct tcp_client {
    tcp_client(const char* server_host, uint16 server_port);
    tcp_client(const tcp_client& c);
    ~tcp_client();

    tcp_client(tcp_client&&) = delete;
    void operator=(const tcp_client& c) = delete;
    void operator=(tcp_client&& c) = delete;

    bool connected() const;
    bool connect(int ms);
    void disconnect();
    void close();

    int recv(void* buf, int n, int ms=-1);
    int recvn(void* buf, int n, int ms=-1);
    int send(const void* buf, int n, int ms=-1);
};
```

- Must be used inside a coroutine.
- If `server_host` is empty, it defaults to `"127.0.0.1"`.
- Copy construction only copies host and port; it does not share the connection.
- The destructor automatically calls `disconnect()`.
- `connect(ms)`: returns `true` if already connected; on failure it has already called `disconnect()` internally and returns `false`; after success it automatically calls `set_tcp_nodelay`.
- `disconnect()` / `close()`: equivalent; repeated calls are safe.
- `recv` / `recvn` / `send`: forward to `co::recv` / `co::recvn` / `co::send`.

{{< hint warning >}}
tcp_client cannot be used by multiple coroutines at the same time; you can put tcp_client into co::pool for reuse.
{{< /hint >}}

### Example

```cpp
#include "co/tcp.h"
#include "co/co.h"
#include "co/print.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    go([] {
        co::tcp_client c("127.0.0.1", 8080);
        if (!c.connect(3000)) {
            co::println("connect failed: ", co::strerror());
            return;
        }

        c.send("hello", 5, 3000);

        char buf[128];
        int n = c.recv(buf, sizeof(buf), 3000);
        if (n > 0) co::println("recv: ", co::string(buf, n));

        c.disconnect();
    });

    co::sleep(5000);
    return 0;
}
```

## tcp_server

```cpp
struct tcp_server {
    tcp_server(const char* ip, uint16 port);
    ~tcp_server();

    tcp_server(const tcp_server&) = delete;
    tcp_server(tcp_server&&) = delete;
    void operator=(const tcp_server&) = delete;
    void operator=(tcp_server&&) = delete;

    using conn_cb_t = std::function<void(sock_t)>;

    tcp_server& on_connection(conn_cb_t&& cb);
    tcp_server& on_connection(const conn_cb_t& cb);

    void start();
    void stop();
    uint32 conn_num();
};
```

- Adopts the one-coroutine-per-connection model.
- If `ip` is empty, it defaults to `"0.0.0.0"`.
- `on_connection` must be called before `start()`; it returns `*this` to support chained calls.
- `start()`: starts a background coroutine for listening; after each successful accept, the callback is executed in an independent coroutine.
- `stop()`: stops listening; semantically it means "stop accepting new connections", not "gracefully shut down the service".
- `conn_num()`: the number of connections currently being processed by callbacks; thread-safe.

### Connection Callback

- Each connection executes the callback in an independent coroutine.
- Before the callback, `keepalive` and `nodelay` are automatically set.
- **After the callback ends, `tcp_server` does not automatically call `co::close(fd)`**; users must close it themselves in the callback.
- After the callback ends, `conn_num` is decremented by 1.

### Example

```cpp
#include "co/tcp.h"
#include "co/co.h"
#include "co/print.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    co::tcp_server s("0.0.0.0", 7777);
    s.on_connection([](sock_t fd) {
        co::println("new connection: ", fd);

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

    s.start();

    co::sleep(60000);

    s.stop();
    return 0;
}
```

## Notes

- When `tcp_client::connect` fails, it has already called `disconnect()` internally; no manual cleanup is needed.
- `tcp_client::send` guarantees that all data is sent on success; the state after an error or timeout is undefined.
- `tcp_server::stop()` only stops listening; it does not close already accepted connections.
- Users must call `co::close(fd)` themselves in the callback.
- `conn_num()` reflects the number of connections currently being processed by callbacks, not the number of established connections.
- When the listening address is a wildcard address (`0.0.0.0` or `::`), `stop()` internally connects to itself using `127.0.0.1` to wake up `accept`.
- The `tcp_server` destructor calls `stop()` first, then releases resources.
