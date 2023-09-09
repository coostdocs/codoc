---
weight: 2
title: "Thread"
---

include: [co/thread.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## Thread

{{< hint warning >}}
Coost v3.0.1 removed the header file `co/thread.h`, and removed the global class `Thread` and `Mutex`, which are nearly the same as `std::thread` and `std::mutex` in C++11. Users can use the std version directly.
{{< /hint >}}


### co::thread_id

```cpp
uint32 thread_id();
```

- Returns the id value of the current thread.



### co::sync_event

{{< hint warning >}}
Coost v3.0.1 removed the global `SyncEvent`, please use `co::sync_event` instead.
{{< /hint >}}


#### constructor

```cpp
explicit sync_event(bool manual_reset=false, bool signaled=false);
```

- Constructor, parameter `manual_reset` indicates whether to manually reset the event to unsynced state, parameter `signaled` indicates whether the initial state is synced.


#### sync_event::reset

```cpp
void reset();
```

- Reset the event to unsynced state.
- When `manual_reset` is true in the constructor, users need manually call this method after calling [wait()](#sync_eventwait) to set the event to unsynced state.


#### sync_event::signal

```cpp
void signal();
```

- Generates a sync signal, set the event to synced state.


#### sync_event::wait

```cpp
1. void wait();
2. bool wait(uint32 ms);
```

- 1, wait until the event becomes synced.
- 2, wait until the event becomes synced or timed out. The parameter `ms` specifies the timeout in milliseconds. Returns true if the event becomes synced, false otherwise.
- When `manual_reset` is false in the constructor, [wait()](#sync_eventwait) will automatically set the event to unsynced state.


#### Code Example

```cpp
#include "co/co.h"

bool manual_reset = false;
co::sync_event ev(manual_reset);

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

int main(int argc, char** argv) {
     std::thread(f1).detach();
     std::thread(f2).detach();
     co::sleep(3000);
     return 0;
}
```



### co::tls

```cpp
template<typename T>
class tls;
```

`co::tls` wraps the system's **thread local** interface.

{{< hint warning >}}
`thread_ptr` was removed in v3.0.1, and `co::tls` was provided instead.
{{< /hint >}}


#### constructor

```cpp
tls();
```

- The constructor, allocate system resources and initialize.


#### tls::get

```cpp
T* get() const;
```

- Return the pointer value owned by the current thread.


#### tls::set

```cpp
void set(T* p);
```

- Set the pointer value owned by the current thread.


#### tls::operator->

```cpp
T* operator->() const;
```

- Returns the pointer value owned by the current thread.


#### tls::operator*

```cpp
T& operator*() const;
```

- Returns a reference to the object pointed to by the pointer owned by the current thread.


#### operator==

```cpp
bool operator==(T* p) const;
```

- Check whether the pointer owned by the current thread is equal to `p`.


#### operator!=

```cpp
bool operator!=(T* p) const;
```

- Check whether the pointer owned by the current thread is not equal to `p`.


#### operator bool

```cpp
explicit operator bool() const;
```

- Returns true if the pointer owned by the current thread is not NULL, false otherwise.
