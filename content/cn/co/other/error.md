---
weight: 2
title: "错误"
---


## 头文件

```cpp
#include "co/error.h"
```

API 在 `co` 命名空间。


## co::error

```cpp
int error();
void error(int e);
```

- `error()` 返回当前错误码。
- `error(e)` 设置当前错误码为 `e`。
- 线程安全。


## co::strerror

```cpp
const char* strerror(int e);
const char* strerror();
```

- 返回 `e` 或当前错误码对应的描述信息。
- 线程安全。
- 返回的字符串不可长时间保存。
