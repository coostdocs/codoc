---
weight: 4
title: "Use third-party network libraries"
---


## Use third-party network libraries in coroutines

There are two ways to use third-party network libraries in coroutines:

- Directly use the blocking APIs of the third-party network library. This is the simplest way, but it relies on the system API hook in coost.
- Use the non-blocking APIs of the third-party network library. In this way, users need to convert the APIs to synchronous manner with [co::io_event](../../concurrency/coroutine/io_event/).



### System API hook

API hook simply intercepts system API requests. If the request is in coroutine and uses a blocking socket, modify the socket to non-blocking mode. When the socket is unreadable or unwritable, use `co::io_event` or the lower-level APIs in coost to wait for the I/O event. When the I/O event arrives, wake up the coroutine and call the system's native socket API to complete the I/O operation.



### Using non-blocking APIs

The following is the `recv` method implemented based on openssl's non-blocking API:

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

The whole process is relatively simple. The underlying socket must be **non-blocking**. When `SSL_read` generates `SSL_ERROR_WANT_READ` error, use `co::io_event` to wait for the read event. When `SSL_ERROR_WANT_WRITE` error occurs, use `co:: io_event` to wait for the write event, when `wait()` returns normally, it means the socket is readable or writable, continue to call `SSL_read` to complete the I/O operation.
