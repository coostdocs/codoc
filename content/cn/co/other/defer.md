---
weight: 2
title: "defer"
---

include: [co/defer.h](https://github.com/idealvin/coost/blob/master/include/co/defer.h).


## defer

**defer** 是 coost 提供的一个宏，它实现了类似 golang 中 defer 的功能。defer 的参数可以是一条或多条语句。

```cpp
void f() {
    void* p = malloc(32);
    defer(free(p));

    defer(
        std::cout << "111" << std::endl;
        std::cout << "222" << std::endl;
    );
    std::cout << "333" << std::endl;
}
```

上面的例子中，`defer` 中的代码将在函数 `f` 结束时执行，因此 `333` 先于 `111` 与 `222` 打印。
