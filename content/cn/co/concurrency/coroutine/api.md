---
weight: 2
title: "APIs"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## Coroutine APIs

{{< hint warning >}}
v3.0 移除了 `co::init`, `co::exit`, `co::stop`。
{{< /hint >}}


### go

```cpp
1. void go(Closure* cb);

2. template<typename F>
   void go(F&& f);

3. template<typename F, typename P>
   void go(F&& f, P&& p);

4. template<typename F, typename T, typename P>
   void go(F&& f, T* t, P&& p);
```

- 此函数用于**创建协程**，与创建线程类似，需要指定一个协程函数。
- 1, 参数 cb 指向一个 Closure 对象，协程启动后会调用 Closure 中的 `run()` 方法。
- 2-4, 将传入的参数打包成一个 Closure，然后调用 1。
- 2, 参数 f 是任意可调用的对象，只要能调用 `f()` 或 `(*f)()` 就行。
- 3, 参数 f 是任意可调用的对象，只要能调用 `f(p)`, `(*f)(p)` 或 `(p->*f)()` 就行。
- 4, 参数 f 是类中带一个参数的方法 `void T::f(P)`，参数 t 是 `T` 类型的指针，参数 p 是方法 f 的参数。
- 实际测试发现，创建 `std::function` 类型的对象开销较大，应尽量少用。

- 示例

```cpp
go(f);             // void f();
go(f, 7);          // void f(int);
go(&T::f, &o);     // void T::f();    T o;
go(&T::f, &o, 3);  // void T::f(int); T o;

// lambda
go([](){
    LOG << "hello co";
});

// std::function
std::function<void()> x(std::bind(f, 7));
go(x);
go(&x); // Ensure that x is alive when the coroutine is running.
```



### DEF_main

这个宏用于定义 main 函数，并将 main 函数中的代码也放到协程中运行。DEF_main 内部已经调用 [flag::parse()](../../../flag/#flagparse) 解析命令行参数，用户无需再次调用。

- 示例

```cpp
DEF_main(argc, argv) {
    go([](){
        LOG << "hello world";
    });
    co::sleep(100);
    return 0;
}
```



### ———————————
### co::coroutine

```cpp
void* coroutine();
```

- 返回当前的 coroutine 指针，若在非协程中调用此函数，则返回值是 NULL。
- 此函数的返回值，可作为 [co::resume()](#coresume) 的参数，用于唤醒协程。



### co::resume

```cpp
void resume(void* p);
```

- 唤醒指定的协程，参数 `p` 是 [co::coroutine()](#cocoroutine) 的返回值。
- 此函数是线程安全的，可在任意地方调用。



### co::yield

```cpp
void yield();
```

- 挂起当前协程，必须在协程中调用。
- 此函数配合 [co::coroutine()](#cocoroutine) 与 [co::resume()](#coresume)，可以手动控制协程的调度，详情参考 [test/yield.cc](https://github.com/idealvin/coost/blob/master/test/yield.cc)。



### ———————————
### co::coroutine_id

```cpp
int coroutine_id();
```

- 返回当前协程的 id，在非协程中调用时，返回值是 -1。



### co::main_sched

```cpp
MainSched* main_sched();
```

- 此函数用于将主线程变成调度线程。
- 用户获取 `MainSched` 指针后，必须在主线程中调用其 `loop()` 方法。

```cpp
#include "co/co.h"
#include "co/cout.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::print("main thread id: ", co::thread_id());

    auto s = co::main_sched();
    for (int i = 0; i < 8; ++i) {
        go([]{
            co::print("thread: ", co::thread_id(), " sched: ", co::sched_id());
        });
    }
    s->loop(); // loop forever
    return 0;  // fake return value
}
```



### co::next_sched

```cpp
Sched* next_sched();
```

- 返回指向下一个 `Sched` 的指针。
- [go(...)](#go) 实际上等价于 `co::next_sched()->go(...)`。

- 示例

```cpp
// 创建在同一个线程中运行的协程
auto s = co::next_sched();
s->go(f1);
s->go(f2);
```

{{< hint warning >}}
v3.0.1 中将 `co::next_scheduler` 重命名为 `co::next_sched`。
{{< /hint >}}



### co::sched

```cpp
Sched* sched();
```

- 返回指向当前协程调度器的指针，调度器与调度线程是一一对应的，如果当前线程不是调度线程，返回值是 NULL。

{{< hint warning >}}
v3.0.1 中将 `co::scheduler` 重命名为 `co::sched`。
{{< /hint >}}



### co::scheds

```cpp
const co::vector<Sched*>& scheds();
```

- 返回协程调度器列表的引用。

{{< hint warning >}}
v3.0.1 中将 `co::schedulers` 重命名为 `co::scheds`。
{{< /hint >}}



### co::sched_id

```cpp
int sched_id();
```

- 返回当前协程调度器的 id，这个值是 0 到 `co::sched_num()-1` 之间的值。如果当前线程不是调度线程，返回值是 -1。

{{< hint warning >}}
v3.0.1 中将 `co::scheduler_id` 重命名为 `co::sched_id`。
{{< /hint >}}



### co::sched_num

```cpp
int sched_num();
```

- 返回协程调度器的数量，此函数常用于实现一些协程安全的数据结构。

- 示例

```cpp
co::vector<T> v(co::sched_num(), 0);

void f() {
    // get object for the current scheduler
    auto& t = v[co::sched_id()];
}

go(f);
```

{{< hint warning >}}
v3.0.1 中将 `co::scheduler_num` 重命名为 `co::sched_num`。
{{< /hint >}}



### co::stop_scheds

```cpp
void stop_scheds();
```

- 停止所有协程调度器，退出所有调度线程。



### 代码示例

```cpp
// print sched id and coroutine id every 3 seconds
#include "co/co.h"
#include "co/cout.h"

void f() {
    while (true) {
        co::print("s: ", co::sched_id(), " c: ", co::coroutine_id());
        co::sleep(3000);
    }
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    for (int i = 0; i < 32; ++i) go(f);
    while (true) sleep::sec(1024);
    return 0;
}
```



### ———————————
### co::sleep

```cpp
void sleep(uint32 ms);
```

- 让当前协程睡一会儿，参数 ms 是时间，单位是毫秒。
- 此函数一般在协程中调用，在非协程中调用相当于 `sleep::ms(ms)`。



### co::timeout

```cpp
bool timeout();
```

- 判断之前的 IO 操作是否超时。用户在调用 `co::recv()` 等带超时时间的函数后，可以调用此函数判断是否超时。
- **此函数必须在协程中调用**。

