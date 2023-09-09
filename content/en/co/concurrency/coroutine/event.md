---
weight: 4
title: "Sync Event"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::event

`co::event` is a synchronization mechanism between coroutines, which is similar to [co::sync_event](../thread/#cosync_event) in threads.

{{< hint info >}}
Starting from v2.0.1, `co::event` can be used in **coroutines and non-coroutines**.
{{< /hint >}}



### constructor

```cpp
1. explicit event(bool manual_reset=false, bool signaled=false);
2. event(event&& e);
3. event(const event& e);
```

- 1, similar to [co::sync_event](../thread/#cosync_event) in threads.
- 2, move constructor.
- 3, copy constructor, only increases the internal reference count by 1.



### reset

```cpp
void reset() const;
```

- Reset the event to unsynced state.



### signal

```cpp
void signal() const;
```

- Generates a sync signal, setting the event to synced state.
- All waiting coroutines or threads will be awakened. If there is no waiting coroutine or thread, the next coroutine or thread that calls the [wait()](#wait) method will return immediately.



### wait

```cpp
1. void wait() const;
2. bool wait(uint32 ms) const;
```

- 1, wait until the event becomes synced.
- 2, Wait until the event becomes synced or times out. The parameter `ms` specifies the timeout in milliseconds. Returns true if the event becomes synced, false otherwise.
- When `manual_reset` in the constructor is false, the event will be automatically set to unsynced state when [wait()](#wait) ends.



### Code Example

```cpp
#include "co/co.h"

int main(int argc, char** argv) {
     flag::parse(argc, argv);

     co::event ev;

     // capture by value,
     // as data on stack may be overwritten by other coroutines.
     go([ev](){
         ev.signal();
     });

     ev.wait(100); // wait for 100 ms
   
     return 0;
}
```
