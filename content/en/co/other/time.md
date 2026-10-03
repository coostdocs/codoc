---
weight: 8
title: "Time"
---

## Header

```cpp
#include "co/time.h"
```

`flag::parse` is not required.

`sleep` is in the `time` namespace; the rest are in the `co` namespace.

## Current Time co::now

```cpp
struct Now {
    static int64 ns();   // nanoseconds, may overflow in 2262
    static int64 us();   // microseconds
    static int64 ms();   // milliseconds
    static co::string str(const char* fmt="%Y-%m-%d %H:%M:%S");
};
```

- `ns()` / `us()` / `ms()` return Unix epoch timestamps (UTC).
- `str()` is based on `strftime` and returns a local time string.

## Monotonic Clock co::mono_time

```cpp
struct MonoTime {
    static int64 ns();
    static int64 us();
    static int64 ms();
};
```

- Based on `std::chrono::steady_clock`.
- Monotonically increasing, suitable for measuring time differences.

## Timer co::timer

```cpp
struct timer {
    timer();
    void restart();
    int64 ns() const;
    int64 us() const;
    int64 ms() const;
};
```

- Records the start point on construction.
- `restart()` resets the start point.
- `ns()` / `us()` / `ms()` return the elapsed time from the start point to now.
- Based on `mono_time`.

## Sleep time::sleep

```cpp
void time::sleep(uint32 ms);
```

- Unit is milliseconds.
- Thread-level sleep, blocks the current thread.
- Cannot be used in a coroutine; use `co::sleep` in coroutines.

## Example

```cpp
#include "co/time.h"
#include "co/print.h"

int main() {
    co::println("now.ns() = ", co::now.ns());
    co::println("now.us() = ", co::now.us());
    co::println("now.ms() = ", co::now.ms());
    co::println("now.str() = ", co::now.str());
    co::println("now.str(%Y%m%d) = ", co::now.str("%Y%m%d"));

    co::println("mono.ns() = ", co::mono_time.ns());
    co::println("mono.us() = ", co::mono_time.us());
    co::println("mono.ms() = ", co::mono_time.ms());

    co::timer t;
    time::sleep(100);
    co::println("elapsed ns = ", t.ns());
    co::println("elapsed us = ", t.us());
    co::println("elapsed ms = ", t.ms());

    t.restart();
    time::sleep(50);
    co::println("after restart, ms = ", t.ms());
    return 0;
}
```
