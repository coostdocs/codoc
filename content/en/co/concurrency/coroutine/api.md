---
weight: 2
title: "APIs"
---


## Coroutine APIs

{{< hint warning >}}
Coost v3.0 removed `co::init`, `co::exit`, `co::stop`.
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

- This function is used to **create a coroutine**, similar to creating a thread, a coroutine function must be specified.
- 1, the parameter `cb` points to a `Closure` object. When the coroutine is started, the `run()` method of `Closure` will be called.
- 2-4, pack the incoming parameters into a `Closure`, and then call the first version.
- 2, the parameter `f` is any runnable object, as long as we can call `f()` or `(*f)()`.
- 3, the parameter `f` is any runnable object, as long as we can call `f(p)`, `(*f)(p)` or `(p->*f)()`.
- 4, the parameter `f` is a method with one parameter in the class `T`, the parameter `t` is a pointer to class `T`, and `p` is the parameter of `f`.
- Creating object of `std::function` is expensive and should be used as little as possible.


- Example

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

This macro is used to define the main function and make code in the main function also run in coroutine. `DEF_main` has already called [flag::parse(argc, argv)](../../flag/#flagparse) to parse the command-line arguments, and users do not need to call it again.

- Example

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

````cpp
void* coroutine();
````

- Get the current coroutine pointer. If it is not called in coroutine, the return value is NULL.
- The return value of this function can be passed to [co::resume()](#coresume) to wake up the coroutine.



### co::resume

````cpp
void resume(void* p);
````

- Wake up the specified coroutine, the parameter `p` is the return value of [co::coroutine()](#cocoroutine).
- It is thread-safe and can be called anywhere.



### co::yield

````cpp
void yield();
````

- Suspend the current coroutine, must be called in coroutine.
- It can be used with [co::coroutine()](#cocoroutine) and [co::resume()](#coresume) to manually control the scheduling of coroutines. See [test/yield.cc](https://github.com/idealvin/coost/blob/master/test/yield.cc) for details.



### ———————————
### co::coroutine_id

```cpp
int coroutine_id();
```

- Returns the id of the current coroutine. If it is called in non-coroutine, the return value is -1.



### co::main_sched

```cpp
MainSched* main_sched();
```

- This function is used to turn the main thread into a scheduling thread.
- Users must call the `loop()` method of `MainSched` in the main thread later.

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

- Returns a pointer points to the next scheduler.
- [go(...)](#go) is actually equivalent to `co::next_sched()->go(...)`.

- Example

```cpp
// create coroutines in the same thread
auto s = co::next_sched();
s->go(f1);
s->go(f2);
```



### co::sched

```cpp
Sched* sched();
```

- Returns a pointer points to the current scheduler. If the current thread is not a scheduling thread, the return value is NULL.



### co::scheds

```cpp
const co::vector<Sched*>& scheds();
```

- Returns a reference to the scheduler list.



### co::sched_id

```cpp
int sched_id();
```

- Returns id of the current scheduler. This value is between 0 and `co::sched_num()-1`. If the current thread is not a scheduling thread, the return value is -1.



### co::sched_num

```cpp
int sched_num();
```

- Returns the number of schedulers.

- Example

```cpp
co::vector<T> v(co::sched_num(), 0);

void f() {
    // get object for the current scheduler
    auto& t = v[co::sched_id()];
}

go(f);
```



### co::stop_scheds

```cpp
void stop_scheds();
```

- Stop all schedulers.



### Code example

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

- Let the current coroutine sleep for a while, the parameter `ms` is time in milliseconds.
- It is usually called in coroutine. Calling it in non-coroutine is equivalent to `sleep::ms(ms)`.



### co::timeout

```cpp
bool timeout();
```

- Check whether the previous IO operation has timed out. When call an API like `co::recv()` with a timeout, we can call this function to determine whether it has timed out.
- **This function must be called in the coroutine**.

