---
weight: 9
title: "定时任务"
---


## 头文件

```cpp
#include "co/tasked.h"
```

API 在 `co` 命名空间，无需 `flag::parse`。


## 概述

- 简单的定时任务调度器，内部一个后台线程运行任务。
- 时间单位为秒，调度粒度为秒。
- 适合低频、冷门任务，不适合高频、精确调度或长时间阻塞的任务。


## 定义

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

- `run_in(c, sec)`：`sec` 秒后执行一次；`sec <= 0` 时立即唤醒。
- `run_every(c, sec)`：每 `sec` 秒执行一次。
- `run_at(c, h, m, s)`：在 `h:m:s` 执行一次；若已过则安排到第二天。
- `run_daily(c, h, m, s)`：每天 `h:m:s` 执行一次。
- `hour` 范围 0-23，`minute` / `second` 范围 0-59，有 `runtime_assert`。
- `stop()`：停止调度器，等待后台线程退出并清空任务队列；多次调用安全。
- 析构时自动 `stop()`。


## 示例

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
