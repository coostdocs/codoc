---
weight: 6
title: "互斥锁"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::mutex

{{< hint info >}}
从 v3.0.1 开始，`co::mutex` 可以在**协程与非协程**中使用。
{{< /hint >}}



### constructor

```cpp
1. mutex();
2. mutex(mutex&& m);
3. mutex(const mutex& m);
```

- 1, 默认构造函数。
- 2, 移动构造函数。
- 3, 拷贝构造函数，仅将内部引用计数加 1。



### lock

```cpp
void lock() const;
```

- 获取锁，阻塞直到获得锁为止。



### try_lock

```cpp
bool try_lock() const;
```

- 获取锁，不会阻塞，成功获取锁时返回 true，否则返回 false。



### unlock

```cpp
void unlock() const;
```

- 释放锁，一般由之前获得锁的协程或线程调用。




## co::mutex_guard

### constructor

```cpp
explicit mutex_guard(co::mutex& m);
explicit mutex_guard(co::mutex* m);
```

- 构造函数，调用 `m.lock()` 获取锁，参数 `m` 是 `co::mutex` 类的引用或指针。



### destructor

```cpp
~mutex_guard();
```

- 析构函数，释放构造函数中获得的锁。



### 代码示例

```cpp
#include "co/co.h"
#include "co/cout.h"

co::mutex g_m;
int g_v = 0;

void f() {
    co::mutex_guard g(g_m);
    ++g_v;
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    go(f);
    go(f);
    f();
    f();
    co::sleep(100);
    co::print("g_v: ", g_v);
    return 0;
}
```
