---
weight: 3
title: "协程"
---


## 头文件

```cpp
#include "co/co.h"
```

API 在 `co` 命名空间。

`go` 已通过 `using co::go;` 引入全局空间，可直接使用。


## 初始化

包含 `co/co.h` 即可，无需显式初始化。

必须在 `main` 开头调用 `flag::parse(argc, argv);`，以启动调度线程。

```cpp
#include "co/co.h"
#include "co/flag.h"
#include "co/print.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    go([] { co::println("coroutine test"); });
    co::sleep(100);
    return 0;
}
```


## 获取调度器与协程信息

```cpp
int      co::sched_num();
sched_t* co::sched();
coro_t*  co::coroutine();
int      co::sched_id();
int      co::coroutine_id();
```

- `sched_num()`：调度器数量，默认 `os::cpunum()`。
- `sched()`：当前调度器，非调度器线程返回 `NULL`。
- `coroutine()`：当前协程。
- `sched_id()`：当前调度器 id，非调度器线程返回 `-1`。
- `coroutine_id()`：当前协程 id，非协程返回 `-1`。


## 启动协程 go

```cpp
void co::go(void (*f)());
template<typename F, typename ...A>
void co::go(F&& f, A&& ...args);
```

- 任意线程可调用，不阻塞。
- `go(f, args...)` 中，`args` 传引用是安全的，**`f` 是 lambda 时，不能按引用捕获协程栈上对象**。
- 协程启动前可能被其他调度器窃取，已启动不可窃取。

{{< hint warning >}}
coost 协程采用共享栈设计，不能通过指针、引用访问另一个协程栈上的内容。
{{< /hint >}}

示例:

```cpp
go(f);                            // void f();
go([]() {co::println("hello");}); // lambda
go(&T::f, (T*)o);                 // void T::f()
```

## sleep 与 timeout

```cpp
void co::sleep(uint32 ms);
bool co::timeout();
```

- `sleep`：协程中让出，非协程中等价 `time::sleep`；`sleep(0)` 也会 yield。
- `timeout`：检查当前协程是否超时，配合 `add_timer` 使用。


## 底层调度接口

### 协程调度

```cpp
void co::yield();
void co::resume(coro_t* c);
```

- `yield`：挂起当前协程，必须在协程中调用。
- `resume`：唤醒协程，`c` 必须来自 `co::coroutine()`，可从任意线程调用。


### 定时器与 IO 事件

```cpp
void co::add_timer(uint32 ms);
void co::add_io_event(sock_t fd, ev_t ev);
void co::del_io_event(sock_t fd, ev_t ev);
void co::del_io_event(sock_t fd);
```

- `add_timer`：给当前协程加超时，需手动调 `yield`；唤醒后用 `timeout()` 判断。
- `add_io_event` / `del_io_event`：注册/删除 socket 读写事件，通常由 socket API 内部调用。


### 栈检查

```cpp
bool co::on_stack(const void* p);
```

- 检查 `p` 所指向内存是否在当前协程栈上。


## 同步机制

### co::mutex

```cpp
struct mutex {
    mutex();
    ~mutex();
    mutex(mutex&& c) noexcept;
    mutex(const mutex& c);
    void operator=(const mutex&) = delete;
    void operator=(mutex&&) = delete;
    void lock() const;
    void unlock() const;
    bool try_lock() const;
    void* _p;
};
```

- 互斥锁，支持协程、非协程。
- 协程中阻塞时让出，非协程中阻塞线程。
- 拷贝构造只增加引用计数。
- 谁上锁谁解锁；跨线程解锁不是良好设计。

#### co::mutex_guard

RAII 锁，构造加锁，析构解锁，不可拷贝、不可移动。


### co::event

```cpp
struct event {
    explicit event(bool manual_reset=false, bool signaled=false);
    ~event();
    event(event&& e) noexcept;
    event(const event& e);
    void operator=(const event&) = delete;
    void operator=(event&&) = delete;
    void wait() const;
    bool wait(uint32 ms) const;
    void notify_one() const;
    void notify_all() const;
    void reset() const;
    void* _p;
};
```

- 同步事件，支持协程、非协程。
- `manual_reset=true`：需手动 `reset()`。
- `manual_reset=false`：`notify_one()` 唤醒一个并自动重置。
- `wait` 阻塞直到被唤醒或超时，不会阻塞调度线程，不会被虚假唤醒，超时返回 `false`。
- 拷贝构造只增加引用计数。


### co::wait_group

```cpp
struct wait_group {
    explicit wait_group(uint32 n);
    wait_group();
    ~wait_group();
    wait_group(wait_group&& wg) noexcept;
    wait_group(const wait_group& wg);
    void operator=(const wait_group&) = delete;
    void operator=(wait_group&&) = delete;
    void add(uint32 n=1) const;
    void done() const;
    void wait() const;
    void* _p;
};
```

- 类似 golang 中 `WaitGroup`。
- 构造可指定初始计数。
- `add(n)` 计数加 `n`，`done()` 计数减 1，减到 0 唤醒所有 `wait()`。
- 拷贝构造只增加引用计数。


## co::pool

```cpp
struct pool {
    using create_cb_t = void* (*)();
    using destroy_cb_t = void (*)(void*);
    pool();
    ~pool();
    pool(create_cb_t c, destroy_cb_t d, uint32 cap=(uint32)-1);
    pool(pool&& p);
    pool(const pool& p);
    void operator=(const pool&) = delete;
    void operator=(pool&&) = delete;
    void* pop() const;
    void push(void* e) const;
    void* _p;
};
```

- 只用于协程中，每个调度器一个池，池中元素不会跨线程共享，使用无需加锁。
- `c` 创建元素，`d` 销毁元素，`cap` 每池最大容量。
- `pop()` 取元素，池空时调用 `c` 创建。
- `push(e)` 放回，超过 `cap` 调用 `d` 销毁，`e` 为 NULL 时忽略。
- 析构时销毁池中元素。
- 拷贝构造只增加引用计数。

{{< hint info >}}
`co::pool` 可用于管理 TCP 连接，大量协程可以复用池中少量连接。
{{< /hint >}}


### co::pool_guard

RAII 守卫，构造 `pop`，析构 `push`，不可拷贝、不可移动。


## 示例

### 启动协程

```cpp
#include "co/co.h"
#include "co/print.h"
#include "co/time.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    go([] {
        co::println("coroutine id: ", co::coroutine_id());
        co::sleep(1000);
        co::println("after sleep");
    });

    go([](int a, int b) {
        co::println("sum = ", a + b);
    }, 1, 2);

    time::sleep(2000);
    return 0;
}
```

### yield / resume

```cpp
co::coro_t* gco = 0;
co::wait_group wg;

void f() {
    co::println("coroutine starts: ", co::coroutine_id());
    gco = co::coroutine();
    co::yield();
    co::println("coroutine ends: ", co::coroutine_id());
    wg.done();
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    wg.add(1);
    go(f);
    time::sleep(1000);
    if (gco) co::resume(gco);
    wg.wait();
    return 0;
}
```

### mutex

```cpp
co::mutex m;
go([m] {
    co::mutex_guard g(m);
    co::print("locked in coroutine\n");
    co::sleep(100);
});
```

{{< hint warning >}}
由于是共享栈，lambda 中需按值捕获。
{{< /hint >}}

### wait_group

```cpp
co::wait_group wg(3);
for (int i = 0; i < 3; ++i) {
    go([wg, i] {
        co::print("task ", i, '\n');
        wg.done();
    });
}
wg.wait();
```

### pool

```cpp
co::pool p(
    []() -> void* { return new co::string("hello"); },
    [](void* e) { delete (co::string*)e; }
);

go([p] {
    co::pool_guard<co::string> g(p);
    co::print("pool string: ", *g, '\n');
});
```

## 协程相关 flag

| flag | 默认值 | 含义 |
|---|---|---|
| `co_sched_num` | 系统 CPU 核数 | 调度器数量 |
| `co_stack_num` | `8` | 每个调度器的栈数量，必须是 2 的 n 次方 |
| `co_stack_size` | `1024 * 1024` | 协程栈大小 |

命令行：

```bash
./app -co_sched_num=4 -co_stack_num=16 -co_stack_size=2m
```

配置文件：

```ini
co_sched_num = 4
co_stack_num = 16
co_stack_size = 2m
```

## 注意事项

- `go` 可在任意线程调用。
- 主线程不是调度器线程，保持简单。
- 调度器默认数量等于 `os::cpunum()`，可用 flag 调整。
- 协程启动前可能被窃取，已启动不可窃取。
- `yield` 必须在协程中调用。
- `resume` 线程安全，可从其他线程唤醒。
- `co::sleep` 在非协程中等价 `time::sleep`。
- `co::sleep(0)` 也会 yield。
- `add_timer` 后需 `yield`，用 `timeout()` 判断超时。
- 共享栈：不能通过指针、引用访问另一个协程栈内容。
- lambda 捕获协程栈上的 `mutex`、`event`、`wait_group` 等需按值捕获。
- `mutex` / `event` / `wait_group` / `pool` 拷贝构造只增加引用计数。
- `pool` 只用于协程中。
- 无需显式关闭调度器。
