---
weight: 2
title: "错误"
---

include: [co/error.h](https://github.com/idealvin/coost/blob/master/include/co/error.h).


## co::error

```cpp
1. int error();
2. void error(int e);
```

- 1, 返回当前的错误码。
- 2, 将当前错误码设置为 `e`。



## co::strerror

```cpp
1. const char* strerror(int e);
2. const char* strerror();
```

- 1, 获取错误码 `e` 的描述信息，线程安全。
- 2, 获取当前错误码的描述信息，线程安全。
