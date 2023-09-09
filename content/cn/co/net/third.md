---
weight: 4
title: "使用三方网络库"
---


## 协程中使用三方网络库

在协程中使用三方网络库有两种方式：

- 直接使用三方网络库的阻塞 API，此方式最简单，依赖于 co 内部的系统 API hook。
- 使用三方网络库的非阻塞 API，此方式需要借助 [co::io_event](../../concurrency/coroutine/io_event/) 将其转换为同步方式。



### 系统 API hook 原理

API hook 简单来说就是拦截系统 API 请求，如果该请求是在协程中，且使用 blocking socket，就将 socket 修改成 non-blocking 模式，当 socket 不可读或写时，利用 `co::io_event` 或 co 中更底层的接口等待 I/O 事件，I/O 事件到来时，再唤醒协程，调用系统原生的 socket API 完成 I/O 操作。



### 使用非阻塞 API

下面是基于 openssl 的非阻塞 API 实现的 recv 方法：

```cpp
int recv(S* s, void* buf, int n, int ms) {
    CHECK(co::sched()) << "must be called in coroutine..";
    int r, e;
    int fd = SSL_get_fd((SSL*)s);
    if (fd < 0) return -1;

    do {
        ERR_clear_error();
        r = SSL_read((SSL*)s, buf, n);
        if (r > 0) return r; // success
        if (r == 0) return 0;
 
        e = SSL_get_error((SSL*)s, r);
        if (e == SSL_ERROR_WANT_READ) {
            co::io_event ev(fd, co::ev_read);
            if (!ev.wait(ms)) return -1;
        } else if (e == SSL_ERROR_WANT_WRITE) {
            co::io_event ev(fd, co::ev_write);
            if (!ev.wait(ms)) return -1;
        } else {
            return r;
        }
    } while (true);
}
```

整个过程比较简单，底层使用 **non-blocking socket**，在 `SSL_read` 产生 `SSL_ERROR_WANT_READ` 错误时，用 `co::io_event` 等待读事件，产生 `SSL_ERROR_WANT_WRITE` 错误时，用 `co::io_event` 等待写事件，`wait()` 正常返回时，表示 socket 可读或可写，继续调用 `SSL_read` 完成 I/O 操作。

一般而言，提供非阻塞 I/O 接口的三方网络库，都可以用与上面类似的方法，将非阻塞 API 转换为同步方式。
