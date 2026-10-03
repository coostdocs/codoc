---
weight: 2
title: "defer"
---

## Header

```cpp
#include "co/defer.h"
```

## defer

- The **defer** macro implements functionality similar to defer in golang.
- The defer argument can be one or more statements.

```cpp
void f() {
    void* p = co::alloc(32);
    defer(co::free(p, 32));

    defer(
        co::println("111");
        co::println("222");
    );
    co::println("333");
}
```

In the above example, the code in `defer` will be executed when the function `f` ends, so `333` is printed before `111` and `222`.
