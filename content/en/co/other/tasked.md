---
weight: 9
title: "Scheduled Tasks"
---

## Header

```cpp
#include "co/tasked.h"
```

The API is in the `co` namespace; `flag::parse` is not required.

## Overview

- A simple scheduled task scheduler with an internal background thread that runs tasks.
- The time unit is seconds, and the scheduling granularity is seconds.
- Suitable for low-frequency, uncommon tasks; not suitable for high-frequency, precise scheduling, or long-blocking tasks.

## Definition

```cpp
struct tasked {
    tasked();
    ~tasked();

    tasked(tasked&& t);
    tasked(const tasked&) = delete;
    void operator=(const tasked&) = delete;
    void operator=(tasked&&) = delete;

    void run_in(closure&& c, int sec);
    void run_every(closure&& c, int sec);
    void run_at(closure&& c, int hour, int minute=0, int second=0);
    void run_daily(closure&& c, int hour=0, int minute=0, int second=0);
    void stop();

    void* _p;
};
```

- `run_in(c, sec)`: executes once after `sec` seconds; when `sec <= 0`, it wakes up immediately.
- `run_every(c, sec)`: executes once every `sec` seconds.
- `run_at(c, h, m, s)`: executes once at `h:m:s`; if the time has already passed, it is scheduled for the next day.
- `run_daily(c, h, m, s)`: executes once every day at `h:m:s`.
- `hour` ranges from 0-23, `minute` / `second` range from 0-59; there are `runtime_assert`s.
- `stop()`: stops the scheduler, waits for the background thread to exit, and clears the task queue; repeated calls are safe.
- Automatically calls `stop()` on destruction.

## Example

```cpp
#include "co/tasked.h"
#include "co/print.h"
#include "co/time.h"

int main() {
    co::tasked t;
    t.run_in([] { co::print("run_in 3s\n"); }, 3);
    t.run_every([] { co::print("run_every 2s\n"); }, 2);
    t.run_at([] { co::print("run_at 10:30:00\n"); }, 10, 30, 0);
    t.run_daily([] { co::print("daily 08:00\n"); }, 8, 0, 0);

    time::sleep(10000);
    t.stop();
    return 0;
}
```
