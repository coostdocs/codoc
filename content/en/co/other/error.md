---
weight: 2
title: "Error"
---

include: [co/error.h](https://github.com/idealvin/coost/blob/master/include/co/error.h).


## co::error

```cpp
1. int error();
2. void error(int e);
```

- 1, returns the current error code.
- 2, set the current error code to `e`.



## co::strerror

```cpp
1. const char* strerror(int e);
2. const char* strerror();
```

- 1, get the description information of the error code `e`, thread-safe.
- 2, get the description information of the current error code, thread-safe.
