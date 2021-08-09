---
weight: 2
title: "defer"
---

include: [co/defer.h](https://github.com/idealvin/co/blob/master/include/co/defer.h).


## defer

`defer` 是 CO 提供的一个宏，它实现了类似 golang 中 defer 的功能。

```cpp
#include "co/defer.h"
#include "co/time.h"
#include "co/log.h"
#include "co/json.h"

void f(const Json& req, Json& res) {
    Timer t;
    LOG << "req: " << req;
    defer(LOG << "res: " << res << ", time elapse: " << t.us() << "us");

    // do something here
}
```

上面的例子中，`defer` 中的代码将在函数 `f` 结束时执行，从而打印出函数的输出及调用时间。
