---
weight: 1
title: "defer"
---

include: [co/defer.h](https://github.com/idealvin/coost/blob/master/include/co/defer.h).


## defer

`defer` is a macro provided by CO, which is similar to defer in golang.

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

In the above example, the code in `defer` will be executed at the end of the function `f`, thus printing the output and time elapse of the function.
