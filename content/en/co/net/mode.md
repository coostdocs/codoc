---
weight: 3
title: "Network programming model"
---


## Coroutine based network programming model

It is easy to write high-concurrency and high-performance network programs with coroutine. Although a coroutine may block, the scheduling thread can quickly switch between a large number of coroutines. Therefore, to achieve high concurrency, we just need to create more coroutines.



### Network model for TCP server

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

- One coroutine for each connection.
- In one coroutine, call `co::accept()` to accept client connections.
- When a connection is accepted, create a new coroutine to handle the connection.
- `on_connection()` is the coroutine function for handling connections, receiving, processing and sending data are performed in a synchronous manner in the coroutine, and we do not need any asynchronous callback.



### Network model for TCP client

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

- Connecting, sending, recving and processing data are performed in a synchronous manner in the coroutine.


In actual applications, [co::pool](../../concurrency/coroutine/pool/) is generally used as a connection pool to avoid creating too many connections:

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
