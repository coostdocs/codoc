---
weight: 2
title: "线程"
---


## 头文件

```cpp
#include "co/thread.h"
```

API 在 `co` 命名空间。


## 线程 id

```cpp
uint32 co::thread_id();
```

- 返回当前线程 id。


## 同步事件

{{< hint info >}}
`std::condition_variable` 允许虚假唤醒(spurious wakeup)，`notify_one` 不保证只唤醒一个等待线程，`wait` 需要带锁，用起来不方便。coost 实现 `co::sync_event` 取代之。
{{< /hint >}}

```cpp
struct sync_event {
    sync_event(bool manual_reset, bool signaled);
    sync_event() : sync_event(false, false) {}
    ~sync_event();

    sync_event(sync_event&& e) noexcept : _p(e._p) { e._p = 0; }

    sync_event(const sync_event&) = delete;
    void operator=(const sync_event&) = delete;
    void operator=(sync_event&&) = delete;

    void notify_one();
    void notify_all();
    void reset();
    void wait();
    bool wait(uint32 ms);

    void* _p;
};
```

- `notify_one` 只唤醒一个等待线程。
- `notify_all` 唤醒所有等待线程。
- `wait` 阻塞直到被唤醒或超时，不会虚假唤醒。
- 支持自动重置、手动重置模式，自动模式下，`wait` 结束时会将信号重置为 `unsignaled` 状态。
- `reset` 用于手动重置模式，将信号重置为 `unsignaled` 状态。
