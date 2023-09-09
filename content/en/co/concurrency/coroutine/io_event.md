---
weight: 5
title: "IO Event"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::_ev_t

```cpp
enum _ev_t {
     ev_read = 1,
     ev_write = 2,
};
```

- Type of I/O event.




## co::io_event

`co::io_event` is used to **convert non-blocking I/O to synchronous mode**. The user performs I/O operations on a **non-blocking socket** in **coroutine**. When the socket is unreadable or unwritable, the user calls the `wait()` method of `co::io_event` to suspend the coroutine and wait for the I/O event; When the socket becomes readable or writable, the scheduling thread will wake up the coroutine again and continue the I/O operation.


### constructor

```cpp
1. io_event(sock_t fd, _ev_t ev);
2. io_event(sock_t fd, int n=0); // for windows only
```

- 1, parameter `fd` is a **non-blocking socket**, parameter `ev` is `ev_read` or `ev_write`. Calling the `wait()` method will wait for the I/O event specified by `ev` on the socket. When `wait()` returns successfully, the user needs to call `recv`, `send` or other API to complete the I/O operation. **On windows, fd must be a TCP socket** (for UDP, it is difficult to simulate the behavior of epoll or kqueue with IOCP).
- 2, for windows only. `fd` can be a UDP socket, but users need to manually call `WSARecvFrom`, `WSASendTo` or other API to send overlapped I/O requests to IOCP, and then call the `wait()` method, when `wait()` returns successfully, it means that IOCP has completed the I/O operation. The specific usage is not detailed here. There are detailed comments in the code. It is recommended to refer directly to the source code of [co::io_event](https://github.com/idealvin/coost/blob/master/include/co/co/io_event.h), and the implementation of [co::accept, co::connect, co::recvfrom, co::sendto](https://github.com/idealvin/coost/blob/master/src/co/sock_win.cc) on windows.



### destructor

```cpp
~io_event();
```

- Removes previously registered I/O events from epoll or kqueue.



### wait

```cpp
bool wait(uint32 ms=-1);
```

- This method waits for the I/O event on the socket. The parameter `ms` is the timeout in milliseconds. The default is -1 and never times out.
- This method blocks until the I/O event arrives, or times out or an error occurs.
- This method returns true on success, false on timeout or error.



### Code Example

```cpp
int recv(sock_t fd, void* buf, int n, int ms) {
     const auto sched = xx::gSched;
     CHECK(sched) << "must be called in coroutine..";

     co::io_event ev(fd, ev_read);
     do {
         int r = (int) __sys_api(recv)(fd, buf, n, 0);
         if (r != -1) return r;

         if (errno == EWOULDBLOCK || errno == EAGAIN) {
             if (!ev.wait(ms)) return -1;
         } else if (errno != EINTR) {
             return -1;
         }
     } while (true);
}
```
