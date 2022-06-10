---
weight: 14
title: "TCP"
---

include: [co/tcp.h](https://github.com/idealvin/co/blob/master/include/co/tcp.h).


## tcp::Connection


`tcp::Connection` is a simple encapsulation of TCP connection, it is designed for TCP server. When SSL is enabled in a TCP server, tcp::Connection will transfer data by SSL. 




### Connection::Connection


```cpp
Connection(int sock);
Connection(void* ssl);
Connection(Connection&& c);
```


- The constructor, `Connection` is created by `tcp::Server`, users do not need to create it manually.
- The first version constructs a normal TCP connection, the second version constructs a TCP connection that support SSL, and the third is a move constructor.




### Connection::~Connection


```cpp
Connection::~Connection();
```


- Destructor, call close() to close the connection.





### Connection::close


```cpp
int close(int ms = 0);
```


- Close the connection. When the parameter ms > 0, close the connection after a certain delay.
- Since v2.0.1, this method can be called anywhere(in coroutine or non-coroutine).





### Connection::recv


```cpp
int recv(void* buf, int n, int ms=-1);
```


- Receive data, similar to [co::recv](../../coroutine/#corecv).
- This method must be called in the coroutine.
- Return > 0 on success, < 0 on timeout or any error, and 0 will be returned if the peer closed the connection.





### Connection::recvn


```cpp
int recvn(void* buf, int n, int ms=-1);
```


- Receive data of specified length, similar to [co::recvn](../../coroutine/#corecvn).
- Return n on success, < 0 on timeout or any error, and 0 will be returned if the peer closed the connection.





### Connection::reset


```cpp
int reset(int ms = 0)
```


- Reset the TCP connection, unlike close(), it will not enter the `TIME_WAIT` state. When the parameter ms > 0, the connection will be reset after a certain delay.
- This method must be called in the I/O thread (usually a coroutine that performs the I/O operations).





### Connection::send


```cpp
int send(const void* buf, int n, int ms=-1);
```


- Send data, similar to [co::send()](../../coroutine/#cosend).
- return n on success, <= 0 on timeout or error.





### Connection::socket


```cpp
int socket() const;
```


- Return the internal socket descriptor, -1 will be returned if the connection was closed.





### Connection::strerror


```cpp
const char* strerror() const;
```


- When an error occurs in a method of Connection, the user can call this method to get the error message.







## tcp::Server


`tcp::Server` is a TCP server based on coroutine. It has the following features:


- Support IPv4 and IPv6.
- Support SSL (openssl is required).
- One coroutine for each client connection.





### Server::Server


```cpp
Server();
```


- The constructor, initialization.



### Server::conn_num

```cpp
uint32 conn_num() const;
```

- Returns number of client connections.



### Server::on_connection


```cpp
Server& on_connection(std::function<void(Connection)>&& f);
Server& on_connection(const std::function<void(Connection)>& f);

template<typename T>
Server& on_connection(void (T::*f)(Connection), T* o);
```


- Set a callback for handling connections.
- In the first 2 versions, the parameter f is a function of type `void f(Connection)`, or a function object of type `std::function<void(Connection)>`.
- In the third version, the parameter f is a method in the class T, and the parameter o is a pointer to type T.
- Since v2.0.2, the parameter of f is an object of tcp::Connection, rather than a pointer, and users do not need to **delete it** any more.
- When the server receives a connection, it will create a new coroutine and call the callback set by this method in the coroutine to handle the connection.



- Example



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

- Set a callback which will be called when the server exits.



### Server::start


```cpp
void start(const char* ip, int port, const char* key=0, const char* ca=0);
```


- Start the TCP server, this method will not block the current thread.
- The parameter ip is the server ip, which can be an IPv4 or IPv6 address, and the parameter port is the server port. 
- The parameter **key** is path of a **PEM** file which stores the SSL private key, and the parameter **ca** is path of a PEM file which stores the SSL certificate. They are NULL by default, and SSL is disabled.
- Starting from v3.0, the server no longer depends on the `tcp::Server` object after it is started.


- Example

```cpp
void f(tcp::Connection conn);
tcp::Server().on_connection(f).start("0.0.0.0", 7788);
```




### Server::exit
```cpp
void exit();
```

- Added since v2.0.2.
- Exit the TCP server, close the listening socket, and no longer receive new connections.
- This method will not close the connections that has been established before.
- If you need to close the previously established connections after the server exits, you can refer to [test/tcp2.cc](https://github.com/idealvin/cocoyaxi/blob/master/test/so/tcp2.cc) or implementations of `http::Server` and `rpc::Server` in co.




## tcp::Client


`tcp::Client` is a TCP client based on coroutine. It has following features:


- Support IPv4 and IPv6.
- Support SSL (openssl is required).
- One client corresponds to one connection.
- It must be used in coroutine.
- It is not coroutine-safe, and it cannot be used by multiple coroutines at the same time.





### Client::Client


```cpp
Client(const char* ip, int port, bool use_ssl=false);
Client(const Client& c);
```


- Constructor. The parameter ip is the ip of the server, which can be a domain name, or an IPv4 or IPv6 address; the parameter port is the server port; the parameter use_ssl indicates whether to enable SSL transmission, the default is false, and SSL is not enabled.
- The second version is the copy constructor, value of ip, port and use_ssl will be copied from another client.
- **The connection is not established in the constructor.** It is generally recommended to check whether the connection has been established before calling recv, send. If not, call the connect() method to establish the connection. It is easy to support auto-reconnection in this way.





### Client::~Client


```cpp
Client::~Client();
```


- Destructor, call the disconnect() method to close the connection.





### Client::close


```cpp
void close();
```


- Close the connection, same as disconnect().





### Client::connect


```cpp
bool connect(int ms);
```


- Establish a connection, the parameter ms is the timeout period in milliseconds.
- This method must be called in the coroutine.
- This method returns true on success, otherwise it returns false. When it fails, the user can use the `strerror()` method to get the error message.





### Client::connected


```cpp
bool connected() const;
```


- Determine whether the connection has been established.





### Client::disconnect


```cpp
void disconnect();
```


- Since v2.0.1, it can be called anywhere(in coroutine or non-coroutine).
- It is safe to call this method multiple times, and it will be called automatically in the destructor.





### Client::recv


```cpp
int recv(void* buf, int n, int ms=-1);
```


- Receive data, similar to [co::recv()](../../coroutine/#corecv).
- This method must be called in the coroutine.
- Return > 0 on success, < 0 on timeout or any error, and 0 will be returned if the peer closed the connection.





### Client::recvn


```cpp
int recvn(void* buf, int n, int ms=-1);
```


- Receive data of specified length, similar to [co::recvn()](../../coroutine/#corecvn).
- This method must be called in the coroutine.
- Return n on success, < 0 on timeout or any error, and 0 will be returned if the peer closed the connection.





### Client::send


```cpp
int send(const void* buf, int n, int ms=-1);
```


- Send data, similar to [co::send()](../../coroutine/#cosend).
- This method must be called in the coroutine.
- return n on success, <= 0 on timeout or error.





### Client::socket


```cpp
int socket() const;
```


- Return the internal socket descriptor.
- When the connection is not established or the connection has been closed, the return value is -1.





### Client::strerror


```cpp
const char* strerror() const;
```


- When an error occurs in a method of tcp::Client, the user can call this method to get the error message.





## TCP server example


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


- The above example implements a simple ping-pong server, when it receives a ping sent by the client, it will reply with a pong.





## TCP client code example


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


- In the above example, we use co::Pool to cache client connections, and different coroutines can share connections in co::Pool.
- co::PoolGuard automatically pop an idle connection from co::Pool during construction, and automatically puts the connection back into co::Pool during destruction.
