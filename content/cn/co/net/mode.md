---
weight: 3
title: "网络编程模式"
---


## 基于协程的网络编程模式

协程可以用同步的方式，实现高并发、高性能的网络程序。协程虽然会阻塞，但调度线程可以在大量的协程间快速切换，因此要实现高并发，只需要创建大量的协程即可。

以 TCP 程序为例，服务端一般采用一个连接一个协程的模式，为每个客户端连接创建新的协程，在协程中处理连接上的数据。客户端没必要一个连接一个协程，一般使用连接池，多个协程共用连接池中的连接。



### 服务端网络模型

```cpp
void on_connection(int fd) {
    while (true) {
        co::recv(fd, ...);  // recv request from client
        process(...);       // process the request
        co::send(fd, ...);  // send response to client
    }
}

void server_fun() {
    while (true) {
        int fd = co::accept(...);
        if (fd != -1) go(on_connection, fd);
    }
}

go(server_fun);
```

- 服务端采用一个连接一个协程的模型。
- 在一个协程中，调用 `co::accept()` 接受客户端连接。
- 有连接到来时，创建一个新的协程，在协程中处理连接上的数据。
- `on_connection()` 是处理连接的协程函数，接收、处理与发送数据，在该协程中以完全同步的方式进行，不需要任何异步回调。



### 客户端网络模型

```cpp
void client_fun() {
    while (true) {
        if (!connected) co::connect(...);  // connect to the server
        co::send(...);                     // send request to the server
        co::recv(...);                     // recv response from the server
        process(...);                      // process the response
        if (over) co::close(...);          // close the connection
    }
}

go(client_fun);
```

- 建立连接，发送、接收、处理数据，在协程中以完全同步的方式进行。


实际应用中，一般使用 [co::pool](../concurrency/coroutine/pool/) 作为连接池，以避免创建过多的连接：

```cpp
co::pool g_p;

void client_fun() {
    while (true) {
        co::pool_guard<Connection> conn(g_p);  // get a idle connection from the pool
        conn->send(...);                       // send request to the server
        conn->recv(...);                       // recv response from the server
        process(...);                          // process the response
        if (over) conn->close(...);            // close the connection
    }
}

go(client_fun);
```

