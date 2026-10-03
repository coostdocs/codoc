---
weight: 2
title: "defer"
---


## 头文件

```cpp
#include "co/defer.h"
```


## defer

- **defer** 宏实现类似 golang 中 defer 的功能。
- defer 参数可以是一条或多条语句。

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

上面的例子中，`defer` 中的代码将在函数 `f` 结束时执行，因此 `333` 先于 `111` 与 `222` 打印。
