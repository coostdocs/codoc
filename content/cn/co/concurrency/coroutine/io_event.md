---
weight: 5
title: "IO 事件"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::_ev_t

```cpp
enum _ev_t {
    ev_read = 1,
    ev_write = 2,
};
```

- I/O 事件类型，`ev_read` 表示读，`ev_write` 表示写。




## co::io_event

`co::io_event` 用于**将非阻塞 I/O 转换为同步方式**。用户在**协程**中对一个 **non-blocking socket** 进行 I/O 操作，当 socket 不可读或不可写时，用户调用 `co::io_event` 的 `wait()` 方法挂起协程，等待 I/O 事件；当 socket 变为可读或可写时，调度线程唤醒该协程，继续 I/O 操作。


### constructor

```cpp
1. io_event(sock_t fd, _ev_t ev);
2. io_event(sock_t fd, int n=0);  // for windows only
```

- 1, **参数 fd 是一个 non-blocking socket**，参数 ev 是 `ev_read` 或 `ev_write` 中的一种。调用 `wait()` 方法会在 socket 上等待 `ev` 指定的 I/O 事件，`wait()` 成功返回时，需要用户调用 `recv`, `send` 等函数完成 I/O 操作。**在 windows 平台，fd 必须是 TCP socket**(对于 UDP，很难用 IOCP 模拟 epoll 或 kqueue 的行为)。
- 2, 仅适用于 windows，`fd` 可以是 UDP socket，但用户需要手动调用 `WSARecvFrom`, `WSASendTo` 等函数向 IOCP 发送 overlapped I/O 请求，然后调用 `wait()` 方法，当 `wait()` 成功返回时，表示 IOCP 已经帮用户完成了 I/O 操作。具体的用法此处不详述，代码中有详细的注释，建议直接参考 [co::io_event 的源码](https://github.com/idealvin/coost/blob/master/include/co/co/io_event.h)，以及 windows 上 [co::accept, co::connect, co::recvfrom, co::sendto 的实现](https://github.com/idealvin/coost/blob/master/src/co/sock_win.cc)。



### destructor

```cpp
~io_event();
```

- 析构函数，从 epoll 或 kqueue 中移除之前注册的 I/O 事件。



### wait

```cpp
bool wait(uint32 ms=-1);
```

- 此方法等待 socket 上的 I/O 事件，参数 `ms` 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此方法阻塞，直到 I/O 事件到来，或者超时、发生错误。
- 此方法成功时返回 true，超时或发生错误时返回 false。



### 代码示例

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

上面的例子是 `co::recv` 的实现，调用原生 recv 方法产生 EWOULDBLOCK 或 EAGAIN 错误时，用 co::io_event 等待读事件，wait() 正常返回时表示 socket 可读，继续调用原生 recv 方法完成读操作。

