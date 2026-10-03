---
weight: 8
title: "时间"
---


## 头文件

```cpp
#include "co/time.h"
```

无需 `flag::parse`。

`sleep` 在 `time` 命名空间，其余在 `co` 命名空间。


## 当前时间 co::now

```cpp
struct Now {
    static int64 ns();   // 纳秒，可能 2262 年溢出
    static int64 us();   // 微秒
    static int64 ms();   // 毫秒
    static co::string str(const char* fmt="%Y-%m-%d %H:%M:%S");
};
```

- `ns()` / `us()` / `ms()` 返回 Unix epoch 时间戳（UTC）。
- `str()` 基于 `strftime`，返回本地时间字符串。


## 单调时钟 co::mono_time

```cpp
struct MonoTime {
    static int64 ns();
    static int64 us();
    static int64 ms();
};
```

- 基于 `std::chrono::steady_clock`。
- 单调递增，适合测时间差。


## 计时器 co::timer

```cpp
struct timer {
    timer();
    void restart();
    int64 ns() const;
    int64 us() const;
    int64 ms() const;
};
```

- 构造时记录起点。
- `restart()` 重置起点。
- `ns()` / `us()` / `ms()` 返回从起点到现在的耗时。
- 基于 `mono_time`。


## 睡眠 time::sleep

```cpp
void time::sleep(uint32 ms);
```

- 单位毫秒。
- 线程级 sleep，会阻塞当前线程。
- 不能在协程中使用，协程中应用 `co::sleep`。


## 示例

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
