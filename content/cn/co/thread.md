---
weight: 10
title: "线程"
---

include: [co/thread.h](https://github.com/idealvin/co/blob/master/include/co/thread.h).


## 线程(Thread)


`Thread` 类是对线程的封装，创建 Thread 对象时，线程就会启动，**线程一旦启动，就会一直运行，直到线程函数退出**。Thread 类并不提供强制退出线程的方法(类似于 pthread_cancel)，这类方法通常不安全，可能对用户数据造成破坏。




### Thread::Thread
```cpp
explicit Thread(co::Closure* cb);

template<typename F>
explicit Thread(F&& f);

template<typename F, typename P>
Thread(F&& f, P&& p);

template<typename F, typename T, typename P>
Thread(F&& f, T* t, P&& p);
```

- 构造函数，Thread 对象创建完，线程就会启动。
- 第 1 个版本，参数 cb 是 Closure 类型的指针，一般不建议用户直接用这个版本。
- 第 2-4 个版本，根据传入的参数构造一个 Closure 对象，然后调用第 1 个版本进行初始化。
- 第 2 个版本，参数 f 是任意可调用的对象，只要能调用 `f()` 或 `(*f)()` 就行。
- 第 3 个版本，参数 f 是任意可调用的对象，只要能调用 `f(p)`, `(*f)(p)` 或 `(p->*f)()` 就行。
- 第 4 个版本，参数 f 是类中带一个参数的方法 `void T::f(P)`，参数 t 是 `T` 类型的指针，p 是方法 f 的参数。



- 示例
```cpp
Thread x([](){});                   // lambda
Thread x(f);                        // void f();
Thread x(f, p);                     // void f(void*);  void* p;
Thread x(f, 7);                     // void f(int v);
Thread x(&T::f, &t);                // void T::f();  T t;
Thread x(&T::f, &t, 7);             // void T::f(int v);  T t;
Thread x(std::bind(&T::f, &t, 7));  // void T::f(int v);  T t;
```




### Thread::~Thread
```cpp
Thread::~Thread();
```

- 析构函数，调用 `join()` 方法，等待线程退出，释放系统资源。 





### Thread::detach
```cpp
void detach();
```

- 让线程独立于 Thread 对象运行，一旦调用此方法，Thread 对象就没什么用了。线程函数退出时，自动释放系统资源。



- 示例
```cpp
void f();
Thread(f).detach();  // run f() in a thread
```




### Thread::join
```cpp
void join();
```

- 调用此方法会阻塞，直到线程函数退出，然后释放系统资源。
- 如果之前已调用过 `detach()` 方法，调用此方法什么也不会发生。






## co::thread_id
```cpp
uint32 thread_id();
```

- 此函数返回当前线程的 id。
- 内部实现中使用 [TLS](https://wiki.osdev.org/Thread_Local_Storage) 保存线程 id，每个线程只需一次系统调用。





## current_thread_id
```cpp
uint32 current_thread_id();
```

- 与 `co::thread_id()` 相同。
- v2.0.2 标记为 deprecated，建议用 `co::thread_id()`。







## 互斥锁(Mutex)


`Mutex` 是多线程编程中常用的一种互斥锁，同一时刻，最多有一个线程占有锁，其他线程必须等待锁被释放。




### Mutex::Mutex
```cpp
Mutex();
```

- 构造函数，分配系统资源并初始化。





### Mutex::~Mutex
```cpp
Mutex::~Mutex();
```

- 析构函数，释放系统资源。





### Mutex::lock
```cpp
void lock();
```

- 获取锁，该方法会阻塞，直到成功获得锁。





### Mutex::try_lock
```cpp
bool try_lock();
```

- 获取锁，不会阻塞，成功获得锁时，返回 true，否则返回 false。





### Mutex::unlock
```cpp
void unlock();
```

- 释放锁，一般由获得锁的线程调用此方法。







## MutexGuard


`MutexGuard` 类用于自动获取、释放 `Mutex` 中的锁，防止用户获取锁后忘记调用 unlock() 方法释放锁。




### MutexGuard::MutexGuard
```cpp
explicit MutexGuard(Mutex& m);
explicit MutexGuard(Mutex* m);
```

- 构造函数，参数 m 是 `Mutex` 类的引用或指针，为指针时，m 不能是 NULL。
- 内部会调用 m 的 `lock()` 方法获取锁。





### MutexGuard::~MutexGuard
```cpp
MutexGuard::~MutexGuard();
```

- 析构函数，释放在构造函数中获取的锁。



- 示例
```cpp
Mutex m;
MutexGuard g(m);
```






## 同步事件(SyncEvent)


`SyncEvent` 是多线程间的一种同步机制，适用于生产者-消费者模型。消费者线程调用 `wait()` 方法等待同步信号，生产者线程则调用 `signal()` 方法产生同步信号。SyncEvent 支持多生产者、多消费者，但实际应用中，单个消费者的情况比较多。




### SyncEvent::SyncEvent
```cpp
explicit SyncEvent(bool manual_reset = false, bool signaled = false);
```

- 构造函数，参数 manual_reset 表示是否手动将同步状态设置成未同步，参数 signaled 表示初始状态是否为同步状态。





### SyncEvent::~SyncEvent
```cpp
SyncEvent::~SyncEvent();
```

- 析构函数，释放系统资源。





### SyncEvent::reset
```cpp
void reset();
```

- 此方法将 SyncEvent 设置成未同步状态。
- 当构造函数中 manual_reset 为 true 时，用户在 wait() 结束时需要手动调用此方法，将 SyncEvent 设置成未同步状态，否则 SyncEvent 可能永远保持同步状态。





### SyncEvent::signal
```cpp
void signal();
```

- 此方法产生同步信号，将 SyncEvent 设置成同步状态。





### SyncEvent::wait
```cpp
void wait();
bool wait(uint32 ms);
```

- 第 1 个版本会一直等待，直到 SyncEvent 变成同步状态。
- 第 2 个版本会等待到 SyncEvent 变成同步状态或超时。参数 ms 指定超时时间，单位为毫秒。
- 第 2 个版本在 SyncEvent 变成同步状态时返回 true，超时则返回 false。
- 当构造函数中 manual_reset 为 false 时，wait() 结束时会自动将 SyncEvent 设置成未同步状态。





### 代码示例
```cpp
bool manual_reset = false;
SyncEvent ev(manual_reset);

void f1() {
    if (!ev.wait(1000)) {
        LOG << "f1: timedout..";
    } else {
        LOG << "f1: event signaled..";
        if (manual_reset) ev.reset();
    }
}

void f2() {
    LOG << "f2: send a signal..";
    ev.signal();
}

Thread(f1).detach();
Thread(f2).detach();
```






## 基于 TLS 的 thread_ptr
```cpp
template <typename T, typename D=std::default_delete<T>>
class thread_ptr;
```
`thread_ptr` 类与 `std::unique_ptr` 类似，但内部使用了 [TLS](https://wiki.osdev.org/Thread_Local_Storage) 机制，每个线程设置并拥有自己的 ptr。




### thread_ptr::thread_ptr
```cpp
thread_ptr();
```

- 构造函数，分配系统资源及初始化。





### thread_ptr::~thread_ptr
```cpp
thread_ptr::~thread_ptr();
```

- 析构函数，delete 各线程私有的 ptr，并释放 TLS 相关系统资源。





### thread_ptr::get
```cpp
T* get() const;
```

- 返回当前线程的 ptr。
- 若该线程之前并未设置过 ptr，则此方法返回 NULL。





### thread_ptr::operator=
```cpp
void operator=(T* p);
```

- 赋值操作，将当前线程的 ptr 设置为 p，等价于 `reset(p)`。





### thread_ptr::operator->
```cpp
T* operator->() const;
```

- 重载 `operator->`，返回当前线程的 ptr。





### thread_ptr::operator*
```cpp
T& operator*() const;
```

- 重载 `operator*`，返回当前线程的 ptr 所指向的 T 类对象的引用。





### thread_ptr::operator==
```cpp
bool operator==(T* p) const;
```

- 重载 `operator==`，判断当前线程的 ptr 是否等于 p。





### thread_ptr::operator!=
```cpp
bool operator!=(T* p) const;
```

- 重载 `operator!=`，判断当前线程的 ptr 是否不等于 p。





### thread_ptr::operator!
```cpp
bool operator!() const;
```

- 重载 `operator!`，判断当前线程的 ptr 是否为 NULL，为 NULL 时返回 true，否则返回 false。





### thread_ptr::operator bool
```cpp
explicit operator bool() const;
```

- 将 thread_ptr 转换为 bool 类型，内部指针不是 NULL 时，返回 true，否则返回 false。





### thread_ptr::release
```cpp
T* release();
```

- 释放当前线程的 ptr。
- 此方法将当前线程的 ptr 设置为 NULL，返回置为 NULL 之前的值。





### thread_ptr::reset
```cpp
void reset(T* p = 0);
```

- 将当前线程的 ptr 重置为 p，p 默认是 0，之前的 ptr 会调用 `D()(x)` delete 掉。





### 代码示例
```cpp
struct T {
    T(int v) : _v(v) {}
    void run() {
        LOG << current_thread_id() << " v: " << _v;
    }
    int _v;
};

thread_ptr<T> pt;

// 每个线程都会设置自己的指针，不同线程互不影响
void f(int v) {
    if (pt == NULL) {
        LOG << "new T(" << v << ")";
        pt.reset(new T(v));
    }
    pt->run();
}

Thread(f, 1).detach(); // 启动线程 1
Thread(f, 2).detach(); // 启动线程 2
```


