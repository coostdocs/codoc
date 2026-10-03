---
weight: 3
title: "TCP"
---

## 头文件

```cpp
#include "co/tcp.h"
```

API 在 `co` 命名空间，同时提供别名：

```cpp
namespace tcp {
using client = co::tcp_client;
using server = co::tcp_server;
}
```

使用前需在 `main` 开头调用 `flag::parse(argc, argv);`。


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

- 必须在协程中使用。
- `server_host` 为空时默认 `"127.0.0.1"`。
- 拷贝构造只复制 host、port，不共享连接。
- 析构时自动 `disconnect()`。
- `connect(ms)`：已连接返回 `true`；失败内部已 `disconnect()`，返回 `false`；成功后自动 `set_tcp_nodelay`。
- `disconnect()` / `close()`：等价，重复调用安全。
- `recv` / `recvn` / `send`：转发到 `co::recv` / `co::recvn` / `co::send`。

{{< hint warning >}}
tcp_client 不能多协程同时使用，可以将 tcp_client 放入 co::pool 复用。
{{< /hint >}}


### 示例

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

- 采用一个连接一个协程(one couroutine per connection)模型。
- `ip` 为空时默认 `"0.0.0.0"`。
- `on_connection` 必须在 `start()` 前调用，返回 `*this` 支持链式调用。
- `start()`：启动后台协程监听，每次 accept 成功后在独立协程中执行回调。
- `stop()`：停止监听，语义上是「停止接受新连接」，不是「优雅关闭服务」。
- `conn_num()`：正在被回调处理的连接数，线程安全。

### 连接回调

- 每个连接在独立协程中执行回调。
- 回调前自动设置 `keepalive` 和 `nodelay`。
- **回调结束后 `tcp_server` 不会自动 `co::close(fd)`**，用户必须在回调中自己关闭。
- 回调结束后 `conn_num` 减 1。

### 示例

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

## 注意事项

- `tcp_client::connect` 失败时内部已 `disconnect()`，无需手动清理。
- `tcp_client::send` 成功时保证全部发送；出错或超时时状态未定义。
- `tcp_server::stop()` 只停止监听，不关闭已接受的连接。
- 用户回调中必须自己 `co::close(fd)`。
- `conn_num()` 反映的是正在被回调处理的连接数，不是已建立的连接数。
- 监听地址为通配地址（`0.0.0.0` 或 `::`）时，`stop()` 内部会用 `127.0.0.1` 连接自己以唤醒 `accept`。
- `tcp_server` 析构会先 `stop()` 再释放资源。
