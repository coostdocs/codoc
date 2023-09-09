---
weight: 3
title: "channel"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::chan

```cpp
template<typename T> class chan;
```

`co::chan` 是一个模板类，它类似于 golang 中的 channel，用于在协程之间传递数据。

{{< hint info >}}
从 v3.0.1 开始，`co::chan` 可在**协程或非协程**中使用，且可存储 `std::string` 等非 POD 类型的值。
{{< /hint >}}



### constructor

```cpp
1. explicit chan(uint32 cap=1, uint32 ms=(uint32)-1);
2. chan(chan&& c);
3. chan(const chan& c);
```

- 1, 参数 cap 是内部队列的最大容量，默认是 1，参数 ms 是读写操作的超时时间，单位为毫秒，默认为 -1，永不超时。
- 2, 移动构造函数。
- 3, 拷贝构造函数，仅将内部引用计数加 1。



### close

```cpp
void close() const;
```

- 关闭 channel，关闭后 channel 不可写，但可读。



### done

```cpp
bool done() const;
```

- 判断读、写操作是否成功完成。



### operator bool

```cpp
explicit operator bool() const;
```

- 若 channel 为关闭状态，返回 false，否则返回 true。



### operator<<

```cpp
chan& operator<<(const T& x) const;
chan& operator<<(T&& x) const;
```

- 写入元素 `x`。
- 此方法会阻塞，直到写入操作完成或超时。可以调用 [done()](#done) 方法判断写操作是否成功完成。
- 若 channel 已关闭，写操作会失败，此方法立即返回。



### operator>>

```cpp
chan& operator>>(T& x) const;
```

- 读取元素。
- 此方法会阻塞，直到读取操作完成或超时。可以调用 [done()](#done) 方法判断读操作是否成功完成。
- 若 channel 已关闭且 channel 中无元素可读时，读操作会失败，此方法立即返回。



### 代码示例

```cpp
#include "co/co.h"
#include "co/cout.h"

void f() {
    co::chan<int> ch;
    go([ch]() { ch << 7; });
    int v = 0;
    ch >> v;
    co::print("v: ", v);
}

void g() {
    co::chan<int> ch(32, 500);
    go([ch]() {
        ch << 7;
        if (!ch.done()) co::print("write to channel timeout..");
    });

    int v = 0;
    ch >> v;
    if (ch.done()) co::print("v: ", v);
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    f();
    g();
    return 0;
}
```
