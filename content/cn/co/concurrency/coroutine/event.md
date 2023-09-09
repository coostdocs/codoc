---
weight: 4
title: "同步事件"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::event

`co::event` 是协程间的一种同步机制，它与线程中的 [co::sync_event](../../thread/#cosync_event) 类似。

{{< hint info >}}
从 v2.0.1 版本开始，`co::event` 可以在**协程与非协程**中使用。
{{< /hint >}}



### constructor

```cpp
1. explicit event(bool manual_reset=false, bool signaled=false);
2. event(event&& e);
3. event(const event& e);
```

- 1, 与线程中的 [co::sync_event](../../thread/#cosync_event) 类似。
- 2, 移动构造函数。
- 3, 拷贝构造函数，仅将内部引用计数加 1。



### reset

```cpp
void reset() const;
```

- 将事件重置为未同步状态。



### signal

```cpp
void signal() const;
```

- 产生同步信号，将事件设置成同步状态。
- 所有 waiting 状态的协程或线程会被唤醒。若当前并没有 waiting 状态的协程或线程，则下一个调用 [wait()](#wait) 方法的协程或线程会立即返回。



### wait

```cpp
1. void wait() const;
2. bool wait(uint32 ms) const;
```

- 1, 等待直到事件变成同步状态。
- 2, 等待直到事件变成同步状态或超时。参数 `ms` 指定超时时间，单位为毫秒。若事件变成同步状态，返回 true，否则返回 false。
- 当构造函数中 `manual_reset` 为 false 时，[wait()](#wait) 结束时会自动将事件设置成未同步状态。



### 代码示例

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

    ev.wait(100);  // wait for 100 ms
   
    return 0;
}
```
