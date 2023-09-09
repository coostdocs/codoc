---
weight: 2
title: "defer"
---

include: [co/defer.h](https://github.com/idealvin/coost/blob/master/include/co/defer.h).


## defer

**defer** is a macro provided by coost, which is similar to defer in golang. It accepts one or more statements as the arguments.

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

In the above example, the code in `defer` will be executed at the end of function `f`, so `333` is printed before `111` and `222`.
