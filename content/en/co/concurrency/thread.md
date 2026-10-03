---
weight: 2
title: "Thread"
---

## Header

```cpp
#include "co/thread.h"
```

The API is in the `co` namespace.

## Thread ID

```cpp
uint32 co::thread_id();
```

- Returns the current thread id.

## Synchronization Event

{{< hint info >}}
`std::condition_variable` allows spurious wakeups, `notify_one` does not guarantee waking only one waiting thread, and `wait` requires a lock, making it inconvenient to use. coost implements `co::sync_event` to replace it.
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

- `notify_one` wakes only one waiting thread.
- `notify_all` wakes all waiting threads.
- `wait` blocks until it is woken up or times out; it does not suffer from spurious wakeups.
- It supports auto-reset and manual-reset modes. In auto-reset mode, `wait` resets the signal to the `unsignaled` state when it returns.
- `reset` is used in manual-reset mode to reset the signal to the `unsignaled` state.
