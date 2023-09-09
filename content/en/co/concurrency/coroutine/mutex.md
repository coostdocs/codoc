---
weight: 6
title: "Mutex Lock"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::mutex

{{< hint info >}}
Starting from v3.0.1, `co::mutex` can be used in **coroutines and non-coroutines**.
{{< /hint >}}



### constructor

```cpp
1. mutex();
2. mutex(mutex&& m);
3. mutex(const mutex& m);
```

- 1, default constructor.
- 2, move constructor.
- 3, copy constructor, only increases the internal reference count by 1.



### lock

```cpp
void lock() const;
```

- Acquire the lock, will block until the lock is acquired.



### try_lock

```cpp
bool try_lock() const;
```

- Acquire the lock, will not block, return true when the lock is successfully acquired, otherwise return false.



### unlock

```cpp
void unlock() const;
```

- Release the lock, usually called by the coroutine or thread that previously obtained the lock.




## co::mutex_guard

### constructor

```cpp
explicit mutex_guard(co::mutex& m);
explicit mutex_guard(co::mutex* m);
```

- Call `m.lock()` to acquire the lock, the parameter `m` is a reference or pointer of the `co::mutex` class.



### destructor

```cpp
~mutex_guard();
```

- Release the lock acquired in the constructor.



### Code Example

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
