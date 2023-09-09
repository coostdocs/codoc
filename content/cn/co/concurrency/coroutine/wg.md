---
weight: 7
title: "wait group"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::wait_group

`co::wait_group` 类似于 golang 中的 `sync.WaitGroup`，可用于等待协程或线程的退出。



### constructor

```cpp
1. explicit wait_group(uint32 n);
2. wait_group();
3. wait_group(wait_group&& wg);
4. wait_group(const wait_group& wg);
```

- 1, 将内部计数器初始化为 `n`。
- 2, 默认构造函数，将内部计数器初始化为 0。
- 3, 移动构造函数。
- 4, 拷贝构造函数，仅将内部引用计数加 1。



### add

```cpp
void add(uint32 n=1) const;
```

- 将内部计数器加 `n`，`n` 默认为 1。



### done

```cpp
void done() const;
```

- 将内部计数器减 1。



### wait

```cpp
void wait() const;
```

- 等待直到内部计数器的值变为 0。



### 代码示例

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

    wg.wait();
    return 0;
}
```
