---
weight: 3
title: "Channel"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::chan

```cpp
template<typename T> class chan;
```

`co::chan` is a template class, which is similar to the channel in golang and is used to pass data between coroutines.

{{< hint info >}}
Since v3.0.1, `co::chan` can be used in **coroutines or non-coroutines**, and can store values ​​of non-POD types such as `std::string`.
{{< /hint >}}



### constructor

```cpp
1. explicit chan(uint32 cap=1, uint32 ms=(uint32)-1);
2. chan(chan&& c);
3. chan(const chan& c);
```

- 1, `cap` is the maximum capacity of the internal queue, the default is 1. Parameter `ms` is the timeout for read and write operations in milliseconds, the default is -1 and never timed out.
- 2, the move constructor.
- 3, the copy constructor, only increases the internal reference count by 1.



### close

```cpp
void close() const;
```

- Close the channel. After closing, the channel cannot be written, but can be read.



### done

```cpp
bool done() const;
```

- Check whether the read or write operation is successfully completed.



### operator bool

```cpp
explicit operator bool() const;
```

- If the channel is closed, returns false, otherwise returns true.



### operator<<

```cpp
chan& operator<<(const T& x) const;
chan& operator<<(T&& x) const;
```

- Write element `x`.
- This method blocks until the write operation completes or times out. You can call the [done()](#done) method to determine whether the write operation is completed successfully.
- If the channel is closed, the write operation will fail and this method will return immediately.



### operator>>

```cpp
chan& operator>>(T& x) const;
```

- Read an element from the channel.
- This method blocks until the read operation completes or times out. You can call the [done()](#done) method to determine whether the read operation is completed successfully.
- If the channel is closed and there is no element in the channel, the read operation will fail and this method will return immediately.



### Code Example

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
