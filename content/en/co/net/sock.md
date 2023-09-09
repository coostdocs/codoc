---
weight: 2
title: "Socket programming"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## Socket APIs

Coost provides commonly used coroutineized socket APIs to support coroutine-based network programming. 

**Most of the APIs are consistent in form with the native socket APIs**, which can reduce the user's learning burden, and users who are familiar with socket programming can easily get started. 

Most of these APIs need to be used in coroutines. When they are blocked on I/O or sleep, the scheduling thread will suspend the current coroutine and switch to other waiting coroutines, and the scheduling thread itself will not block. With these APIs, users can easily implement **high-concurrency and high-performance network programs**.



### Terminology convention

**Blocking**

When describing some socket APIs (accept, recv, etc.) in coost, the term blocking is used. The document says that they will block, which means that the current coroutine will block, but the scheduling thread will not (it can switch to other coroutines to run). From the user's point of view, they saw the coroutines which may block. In fact, these APIs use **non-blocking socket** internally, which does not really block, but when the socket is not readable or writable, the scheduling thread will suspend the current coroutine. When the socket becomes readable or writable, the scheduling thread will awaken the coroutine again and continue the I/O operation. 


**non-blocking socket**

The socket APIs in coost must use **non-blocking socket**. On windows, socket must also support [overlapped I/O](https://support.microsoft.com/en-us/help/181611/socket-overlapped-io-versus-blocking-nonblocking-mode), which is supported by default for sockets created with win32 API, users generally no need to worry about it. For narrative convenience, when non-blocking socket is mentioned in the document, it also means that it supports overlapped I/O on windows.



### co::socket

```cpp
1. sock_t socket(int domain, int type, int proto);
2. sock_t tcp_socket(int domain=AF_INET);
3. sock_t udp_socket(int domain=AF_INET);
```

- Create a socket.
- 1, the form is exactly the same as the native API. You can use `man socket` to see the details on linux.
- 2, create a TCP socket.
- 3, create a UDP socket.
- The parameter `domain` is usually AF_INET or AF_INET6, the former means ipv4 and the latter means ipv6.
- **These functions return a non-blocking socket**. When an error occurs, the return value is -1, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::accept

```cpp
sock_t accept(sock_t fd, void* addr, int* addrlen);
```

- Receive the client connection on the specified socket, the parameter fd is a non-blocking socket, and the parameters addr and addrlen are used to receive the client's address information. The initial value of `*addrlen` is the length of the buffer pointed to by addr. If the user does not need the client address information, addr and addrlen should be set to NULL.
- This function **must be called in coroutine**.
- This function will block until a new connection comes in or an error occurs.
- This function **returns a non-blocking socket** on success, and returns -1 when an error occurs, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::bind

```cpp
int bind(sock_t fd, const void* addr, int addrlen);
```

- Bind the ip address to the socket, the parameters addr and addrlen are the address information, which is the same as the native API.
- This function returns 0 on success, otherwise returns -1, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::close

```cpp
int close(sock_t fd, int ms=0);
```

- Close the socket.
- In coost v2.0.0 or before, a socket must be closed in the same thread that performed the I/O operation. Since v2.0.1, a socket can be closed anywhere.
- When `ms > 0`, it will call `co::sleep(ms)` to suspend the current coroutine for a period of time, and then close the socket. 
- The **EINTR** signal has been processed internally in this function, and the user does not need to consider it.
- This function returns 0 on success, otherwise it returns -1, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::connect

```cpp
int connect(sock_t fd, const void* addr, int addrlen, int ms=-1);
```

- Create a connection to the specified address on the specified socket, parameter `fd` must be non-blocking, `ms` is the timeout period in milliseconds, the default is -1, which will never time out.
- This function **must be called in coroutine**.
- This function will block until the connection is completed, or timeout or an error occurs.
- This function returns 0 on success, and returns -1 on timeout or an error occurs. The user can call [co::timeout()](../../concurrency/coroutine/api/#cotimeout) to check whether it has timed out, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::listen

```cpp
int listen(sock_t fd, int backlog=1024);
```

- Listenning on the specified socket.
- This function returns 0 on success, otherwise it returns -1, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::recv

```cpp
int recv(sock_t fd, void* buf, int n, int ms=-1);
```

- Receive data on the specified socket, parameter `fd` must be non-blocking, `buf` is the buffer to receive the data, `n` is the buffer length, and `ms` is the timeout period in milliseconds, the default is -1, never time out.
- This function **must be called in coroutine**.
- On Windows, this function only works with TCP-like stream socket.
- This function will block until any data comes in, or timeout or any error occurs.
- This function returns length of the data received (may be less than `n`) on success, returns 0 when the peer closes the connection, returns -1 when timeout or an error occurs, and users can call [co::timeout()](../../concurrency/coroutine/api/#cotimeout) to check whether it has timed out, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::recvn

```cpp
int recvn(sock_t fd, void* buf, int n, int ms=-1);
```

- Receive `n` bytes on the specified socket, `fd` must be non-blocking, `ms` is the timeout period in milliseconds, the default is -1, never timeout.
- This function **must be called in coroutine**.
- This function will block until all `n` bytes of data are received, or timeout or an error occurs.
- This function returns `n` on success, returns 0 when the peer closes the connection, and returns -1 when timeout or an error occurs. The user can call [co::timeout()](../../concurrency/coroutine/api/#cotimeout) to check whether it has timed out, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::recvfrom

```cpp
int recvfrom(sock_t fd, void* buf, int n, void* src_addr, int* addrlen, int ms=-1);
```

- Similar to [co::recv](#corecv), except that the parameters `src_addr` and `addrlen` can be used to receive the source address information. The initial value of `*addrlen` is the length of the buffer pointed to by `src_addr`. If the user does not need the source address information, `addr` and `addrlen` should be set to NULL .
- **It is recommended to use this function to receive UDP data only**.



### co::send

```cpp
int send(sock_t fd, const void* buf, int n, int ms=-1);
```

- Send data to the specified socket, parameter `fd` must be non-blocking, `ms` is the timeout period in milliseconds, the default is -1, which will never time out.
- This function **must be called in coroutine**.
- On Windows, this function only works with TCP-like stream socket.
- This function will block until all `n` bytes of data are sent, or timeout or an error occurs.
- This function returns `n` on success, and returns -1 on timeout or an error occurs. The user can call [co::timeout()](../../concurrency/coroutine/api/#cotimeout) to check whether it has timed out, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### co::sendto

```cpp
int sendto(sock_t fd, const void* buf, int n, const void* dst_addr, int addrlen, int ms=-1);
```

- Send data to the specified address. When `dst_addr` is NULL and `addrlen` is 0, it is equivalent to [co::send](#cosend).
- **It is recommended to use this function to send UDP data only**.
- When `fd` is a UDP socket, the maximum of `n` is 65507.



### co::shutdown

```cpp
int shutdown(sock_t fd, char c='b');
```

- This function is generally used to half-close the socket. The parameter `c` is a hint, `'r'` for read, `'w'` for write, the default is `'b'` for both read and write.
- It is better to call this function in the same thread that performed the I/O operation.
- This function returns 0 on success, otherwise it returns -1, and [co::error()](../../other/error/#coerror) can be called to get the error code.



### ———————————
### co::getsockopt

```cpp
int getsockopt(sock_t fd, int lv, int opt, void* optval, int* optlen);
```

- Get socket option information, which is exactly the same as native API, **man getsockopt** for details.



### co::setsockopt

```cpp
int setsockopt(sock_t fd, int lv, int opt, const void* optval, int optlen);
```

- Set the socket option information, which is exactly the same as the native API, **man setsockopt** for details.



### co::set_nonblock

```cpp
void set_nonblock(sock_t fd);
```

- Set **O_NONBLOCK** option for the socket.



### co::set_reuseaddr

```cpp
void set_reuseaddr(sock_t fd);
```

- Set **SO_REUSEADDR** option for the socket. 
- Generally, the listening socket of a server needs to set this option, or [bind](#cobind) may fail when the server restarts.



### co::set_recv_buffer_size

```cpp
void set_recv_buffer_size(sock_t fd, int n);
```

- Set receiving buffer size of the socket. This function must be called before the socket is connected.



### co::set_send_buffer_size

```cpp
void set_send_buffer_size(sock_t fd, int n);
```

- Set sending buffer size of the socket. This function must be called before the socket is connected.



### co::set_tcp_keepalive

```cpp
void set_tcp_keepalive(sock_t fd);
```

- Set **SO_KEEPALIVE** option for the socket.



### co::set_tcp_nodelay

```cpp
void set_tcp_nodelay(sock_t fd);
```

- Set **TCP_NODELAY** option for the socket.



### co::reset_tcp_socket

```cpp
int reset_tcp_socket(sock_t fd, int ms=0);
```

- Reset a TCP connection, similar to [co::close](#coclose), but the caller will not enter the TIME_WAIT state.
- Generally, only the server side will call this function to close a client connection without entering the TIME_WAIT state.



### ———————————
### co::addr2str

```cpp
1. fastring addr2str(const struct sockaddr_in* addr);
2. fastring addr2str(const struct sockaddr_in6* addr);
3. fastring addr2str(const void* addr, int len);
```

- Convert the socket address into a string in the form of `"ip:port"`.
- 1, for ipv4 addresses, 2 for ipv6 addresses. The third one calls version 1 or version 2 according to `addrlen`.



### co::init_addr

```cpp
1. bool init_addr(struct sockaddr_in* addr, const char* ip, int port);
2. bool init_addr(struct sockaddr_in6* addr, const char* ip, int port);
```

- Initialize the sockaddr structure with ip and port.
- 1, for ipv4 addresses, 2 for ipv6 addresses.

- Example

```cpp
union {
    struct sockaddr_in v4;
    struct sockaddr_in6 v6;
} addr;

co::init_ip_addr(&addr.v4, "127.0.0.1", 7777);
co::init_ip_addr(&addr.v6, "::", 7777);
```



### co::init_ip_addr

```cpp
bool init_ip_addr(struct sockaddr_in* addr, const char* ip, int port);
bool init_ip_addr(struct sockaddr_in6* addr, const char* ip, int port);
```

- Deprecated since v3.0.1, use [co::init_addr](#coinit_addr) instead.



### co::peer

```cpp
fastring peer(sock_t fd);
```

- Get the address information of the peer. The return value is a string in the form of `"ip:port"`.

