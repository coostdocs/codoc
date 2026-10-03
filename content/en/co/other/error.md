---
weight: 2
title: "Error"
---

## Header

```cpp
#include "co/error.h"
```

The API is in the `co` namespace.

## co::error

```cpp
int error();
void error(int e);
```

- `error()` returns the current error code.
- `error(e)` sets the current error code to `e`.
- Thread-safe.

## co::strerror

```cpp
const char* strerror(int e);
const char* strerror();
```

- Returns the description corresponding to `e` or the current error code.
- Thread-safe.
- The returned string must not be stored for a long time.
