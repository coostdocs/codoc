---
weight: 3
title: "Coroutine"
---

## Header

```cpp
#include "co/co.h"
```

The API is in the `co` namespace.

`go` has been introduced into the global namespace via `using co::go;` and can be used directly.

## Initialization

Just include `co/co.h`; no explicit initialization is needed.

You must call `flag::parse(argc, argv);` at the beginning of `main` to start the scheduling threads.

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

## Getting Scheduler and Coroutine Information

```cpp
int      co::sched_num();
sched_t* co::sched();
coro_t*  co::coroutine();
int      co::sched_id();
int      co::coroutine_id();
```

- `sched_num()`: number of schedulers, defaults to `os::cpunum()`.
- `sched()`: current scheduler; returns `NULL` for non-scheduler threads.
- `coroutine()`: current coroutine.
- `sched_id()`: current scheduler id; returns `-1` for non-scheduler threads.
- `coroutine_id()`: current coroutine id; returns `-1` for non-coroutine contexts.

## Starting a Coroutine: go

```cpp
void co::go(void (*f)());
template<typename F, typename ...A>
void co::go(F&& f, A&& ...args);
```

- Can be called from any thread; does not block.
- In `go(f, args...)`, passing `args` by reference is safe; **if `f` is a lambda, objects on the coroutine stack must not be captured by reference**.
- A coroutine may be stolen by another scheduler before it starts; once started, it cannot be stolen.

{{< hint warning >}}
coost coroutines use a shared stack design; you cannot access the contents of another coroutine's stack through pointers or references.
{{< /hint >}}

Example:

```cpp
go(f);                            // void f();
go([]() {co::println("hello");}); // lambda
go(&T::f, (T*)o);                 // void T::f()
```

## sleep and timeout

```cpp
void co::sleep(uint32 ms);
bool co::timeout();
```

- `sleep`: yields in a coroutine; equivalent to `time::sleep` in a non-coroutine context; `sleep(0)` also yields.
- `timeout`: checks whether the current coroutine has timed out; used together with `add_timer`.

## Low-Level Scheduling Interfaces

### Coroutine Scheduling

```cpp
void co::yield();
void co::resume(coro_t* c);
```

- `yield`: suspends the current coroutine; must be called inside a coroutine.
- `resume`: wakes up a coroutine; `c` must come from `co::coroutine()`; can be called from any thread.

### Timers and IO Events

```cpp
void co::add_timer(uint32 ms);
void co::add_io_event(sock_t fd, ev_t ev);
void co::del_io_event(sock_t fd, ev_t ev);
void co::del_io_event(sock_t fd);
```

- `add_timer`: adds a timeout to the current coroutine; you need to call `yield` manually; after waking up, use `timeout()` to check.
- `add_io_event` / `del_io_event`: register/remove socket read/write events; usually called internally by socket APIs.

### Stack Check

```cpp
bool co::on_stack(const void* p);
```

- Checks whether the memory pointed to by `p` is on the current coroutine's stack.

## Synchronization Mechanisms

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

- Mutex; supports coroutines and non-coroutines.
- When blocking in a coroutine, it yields; when blocking in a non-coroutine context, it blocks the thread.
- Copy construction only increases the reference count.
- Whoever locks should unlock; unlocking across threads is not a good design.

#### co::mutex_guard

RAII lock; locks on construction, unlocks on destruction; non-copyable and non-movable.

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

- Synchronization event; supports coroutines and non-coroutines.
- `manual_reset=true`: requires manual `reset()`.
- `manual_reset=false`: `notify_one()` wakes one and automatically resets.
- `wait` blocks until it is woken up or times out; it does not block the scheduling thread and is not subject to spurious wakeups; returns `false` on timeout.
- Copy construction only increases the reference count.

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

- Similar to `WaitGroup` in golang.
- The constructor can specify an initial count.
- `add(n)` increments the count by `n`; `done()` decrements the count by 1; when it reaches 0, all `wait()` calls are woken up.
- Copy construction only increases the reference count.

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

- Used only in coroutines; each scheduler has one pool; elements in the pool are not shared across threads, so no locking is needed.
- `c` creates elements, `d` destroys elements, `cap` is the maximum capacity per pool.
- `pop()` takes an element; if the pool is empty, calls `c` to create one.
- `push(e)` puts it back; if it exceeds `cap`, calls `d` to destroy it; if `e` is NULL, it is ignored.
- Elements in the pool are destroyed at destruction time.
- Copy construction only increases the reference count.

{{< hint info >}}
`co::pool` can be used to manage TCP connections; a large number of coroutines can reuse a small number of connections in the pool.
{{< /hint >}}

### co::pool_guard

RAII guard; calls `pop` on construction and `push` on destruction; non-copyable and non-movable.

## Examples

### Starting Coroutines

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
Because of the shared stack, lambdas must capture by value.
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

## Coroutine-Related Flags

| flag | Default | Meaning |
|---|---|---|
| `co_sched_num` | Number of system CPU cores | Number of schedulers |
| `co_stack_num` | `8` | Number of stacks per scheduler; must be a power of 2 |
| `co_stack_size` | `1024 * 1024` | Coroutine stack size |

Command line:

```bash
./app -co_sched_num=4 -co_stack_num=16 -co_stack_size=2m
```

Configuration file:

```ini
co_sched_num = 4
co_stack_num = 16
co_stack_size = 2m
```

## Notes

- `go` can be called from any thread.
- The main thread is not a scheduler thread; keep it simple.
- The default number of schedulers equals `os::cpunum()`; it can be adjusted with flags.
- A coroutine may be stolen before it starts; once started, it cannot be stolen.
- `yield` must be called inside a coroutine.
- `resume` is thread-safe and can wake up a coroutine from another thread.
- `co::sleep` is equivalent to `time::sleep` in a non-coroutine context.
- `co::sleep(0)` also yields.
- After `add_timer`, you need to `yield`, and use `timeout()` to check for timeout.
- Shared stack: you cannot access the contents of another coroutine's stack through pointers or references.
- Capturing coroutine-stack `mutex`, `event`, `wait_group`, etc. in lambdas requires capturing by value.
- Copy construction of `mutex` / `event` / `wait_group` / `pool` only increases the reference count.
- `pool` is used only in coroutines.
- No explicit shutdown of schedulers is needed.
