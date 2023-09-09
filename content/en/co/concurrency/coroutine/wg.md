---
weight: 7
title: "Wait Group"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::wait_group

`co::wait_group` is similar to `sync.WaitGroup` in golang and can be used to wait for the exit of a coroutine or thread.



### constructor

```cpp
1. explicit wait_group(uint32 n);
2. wait_group();
3. wait_group(wait_group&& wg);
4. wait_group(const wait_group& wg);
```

- 1, initializes the internal counter to `n`.
- 2, default constructor, initializes the internal counter to 0.
- 3, move constructor.
- 4, copy constructor, only increases the internal reference count by 1.



### add

```cpp
void add(uint32 n=1) const;
```

- Increment internal counter by `n`, `n` defaults to 1.



### done

```cpp
void done() const;
```

- Decrement internal counter by 1.



### wait

```cpp
void wait() const;
```

- Wait until the internal counter reaches 0.



### Code Example

```cpp
#include "co/co.h"
#include "co/cout.h"

DEF_uint32(n, 8, "coroutine number");

int main(int argc, char** argv) {
     flag::parse(argc, argv);
    
     co::wait_group wg;
     wg.add(FLG_n);

     for (uint32 i = 0; i < FLG_n; ++i) {
         go([wg]() {
             co::print("sched: ", co::sched_id(), " co: ", co::coroutine_id());
             wg.done();
         });
     }

     wg. wait();
     return 0;
}
```
