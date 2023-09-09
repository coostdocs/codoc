---
weight: 2
title: "线程"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## 线程

{{< hint warning >}}
v3.0.1 删除了 `co/thread.h` 头文件，移除了全局的 `Thread` 与 `Mutex` 类，其功能与 C++11 中的 `std::thread` 与 `std::mutex` 一样，用户可以直接使用 std 版本。
{{< /hint >}}


### co::thread_id

```cpp
uint32 thread_id();
```

- 返回当前线程的 id 值。



### co::sync_event

同步事件是多线程间的一种同步机制，适用于生产者-消费者模型。消费者线程调用 `wait()` 方法等待同步信号，生产者线程则调用 `signal()` 方法产生同步信号。`co::sync_event` 支持多生产者、多消费者，但实际应用中，单个消费者的情况比较多。

{{< hint warning >}}
v3.0.1 移除了全局范围的 `SyncEvent`，请使用 `co::sync_event` 替代之。
{{< /hint >}}


#### constructor

```cpp
explicit sync_event(bool manual_reset=false, bool signaled=false);
```

- 构造函数，参数 `manual_reset` 表示是否手动将同步状态设置成未同步，参数 `signaled` 表示初始状态是否为同步状态。


#### sync_event::reset

```cpp
void reset();
```

- 将事件设置成未同步状态。
- 当构造函数中 `manual_reset` 为 true 时，用户在调用 [wait()](#sync_eventwait) 后需要手动调用此方法，将事件设置成未同步状态。


#### sync_event::signal

```cpp
void signal();
```

- 产生同步信号，将事件设置成同步状态。


#### sync_event::wait

```cpp
1. void wait();
2. bool wait(uint32 ms);
```

- 1, 一直等待直到事件变成同步状态。
- 2, 等待直到事件变成同步状态或超时。参数 `ms` 指定超时时间，单位为毫秒。若事件变成同步状态，返回 true，否则返回 false。
- 当构造函数中 `manual_reset` 为 false 时，[wait()](#sync_eventwait) 结束时会自动将事件设置成未同步状态。


####  代码示例

```cpp
#include "co/co.h"

bool manual_reset = false;
co::sync_event ev(manual_reset);

void f1() {
    if (!ev.wait(1000)) {
        LOG << "f1: timedout..";
    } else {
        LOG << "f1: event signaled..";
        if (manual_reset) ev.reset();
    }
}

void f2() {
    LOG << "f2: send a signal..";
    ev.signal();
}

int main(int argc, char** argv) {
    std::thread(f1).detach();
    std::thread(f2).detach();
    co::sleep(3000);
    return 0;
}
```



### co::tls

```cpp
template<typename T>
class tls;
```

`co::tls` 对系统的 **thread local** 接口进行封装。

{{< hint warning >}}
v3.0.1 中移除了 `thread_ptr`，取而代之提供了 `co::tls`。
{{< /hint >}}


#### constructor

```cpp
tls();
```

- 构造函数，分配系统资源及初始化。


#### tls::get

```cpp
T* get() const;
```

- 返回当前线程拥有的指针值。


#### tls::set

```cpp
void set(T* p);
```

- 设置当前线程拥有的指针值。


#### tls::operator->

```cpp
T* operator->() const;
```

- 返回当前线程拥有的指针值。


#### tls::operator*

```cpp
T& operator*() const;
```

- 返回当前线程拥有指针所指向对象的引用。


#### operator==

```cpp
bool operator==(T* p) const;
```

- 判断当前线程所拥有的指针是否等于 `p`。


#### operator!=

```cpp
bool operator!=(T* p) const;
```

- 判断当前线程所拥有的指针是否不等于 `p`。


#### operator bool

```cpp
explicit operator bool() const;
```

- 当前线程所拥有的指针不是 NULL 时，返回 true，否则返回 false。
