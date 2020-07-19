---
title: "高效获取当前线程的 id"
date: 2018-09-28T17:25:11+08:00
author: Alvin
keywords: thread,thread id,syscall,GetCurrentThreadId
---

## 不同平台获取当前线程 id 的方法

* **Windows**

可以利用微软提供的 api，轻松获取当前线程的 id：

```cpp
int id = GetCurrentThreadId();
```

* **Linux**

Linux 平台一般用的是 [pthread](https://en.wikipedia.org/wiki/POSIX_Threads) 线程库，但它并不提供获取线程内部 id 的方法。可以通过[系统调用](https://en.wikipedia.org/wiki/System_call)得到当前线程的 id：

```cpp
#include <unistd.h>       // for syscall()
#include <sys/syscall.h>  // for SYS_xxx definitions

int id = syscall(SYS_gettid);
```

* **Mac**

Mac 平台也可以用 `syscall` 获取当前线程的 id，与 Linux 稍有区别：

```cpp
#include <unistd.h>       // for syscall()
#include <sys/syscall.h>  // for SYS_xxx definitions

int id = syscall(SYS_thread_selfid); // for mac os x
```

不过在 mac os x 10.12 之后，syscall 被标记为 **deprecated**，所以最好还是用下面的方法取代 syscall：

```cpp
#include <pthread.h>

uint64_t id;
pthread_threadid_np(0, &id); // non-posix, supported by BSD
```

## 基于 TLS 的优化

系统调用会在用户态与内核态之间来回切换，相对比较耗时。为了避免频繁的系统调用，可以用 [TLS](https://en.wikipedia.org/wiki/Thread-local_storage) 优化，每个线程只需一次系统调用：
```cpp
inline int tls_get_tid() {
    static __thread int id = 0;
    if (id != 0) return id;
    id = syscall(SYS_gettid);
    return id;
}
```

下面是一段简单的测试代码：
```cpp
inline int sys_get_tid() {
    return syscall(SYS_gettid);
}

void fsys() {
    int v = 0;
    Timer t;
    for (int i = 0; i < 1000000; i++) {
        v = sys_get_tid();
    }

    int64 us = t.us();
    cout << "fsys use " << us << "us" << "  id: " << v << endl;
}

void ftls() {
    int v = 0;
    Timer t;
    for (int i = 0; i < 1000000; i++) {
        v = tls_get_tid();
    }

    int64 us = t.us();
    cout << "ftls use " << us << "us" << "  id: " << v << endl;
}
```

在 Linux 系统编译执行结果如下：
```sh
# ./xx
fsys use 299251us  id: 71
ftls use 2675us  id: 71
```

可以看到 `TLS` 版本性能提升了将近 100 倍，效果非常明显。

我在 windows 上也进行了类似的测试，结果表明 TLS 对性能没什么影响，可能 windows 的 api 内部就是用 TLS 机制实现的。
