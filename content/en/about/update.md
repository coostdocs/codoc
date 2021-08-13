---
weight: 4
title: "Changelog"
---


## v2.0.1


### Documents

- English: [github](https://idealvin.github.io/en/about/co/) [gitee](https://idealvin.gitee.io/en/about/co/)
- 中文: [github](https://idealvin.github.io/cn/about/co/) [gitee](https://idealvin.gitee.io/cn/about/co/)



### New features

- xrepo
```sh
xrepo install -f "openssl=true,libcurl=true" co
```

- vcpkg
```sh
vcpkg install co:x64-windows

# http & ssl support
vcpkg install co[libcurl,openssl]:x64-windows
```

- defer (similar to defer in golang)
```cpp
#include "co/defer.h"
Timer t;
defer(LOG << "time elapse: " << t.us() << "us");
```

- channel for coroutine (similar to channel in golang)
```cpp
#include "co/co.h"

DEF_main(argc, argv) {
    co::Chan<int> ch;
    go([ch]() {
        ch << 7;
    });

    int v = 0;
    ch >> v;
    LOG << "v: "<< v;

    return 0;
}
```

- waitgroup (similar to sync.WaitGroup in golang)
```cpp
#include "co/co.h"

DEF_main(argc, argv) {
    FLG_cout = true;

    co::WaitGroup wg;
    wg.add(8);

    for (int i = 0; i <8; ++i) {
        go([wg]() {
            LOG << "co: "<< co::coroutine_id();
            wg.done();
        });
    }

    wg.wait();
    return 0;
}
```

- Coroutine hook for windows.

- Create coroutines in specified scheduler(thread).
```cpp
auto s = co::next_scheduler();
s->go(f1);
s->go(f2);
```

- Create coroutine in all schedulers.
```cpp
auto& s = co::all_schedulers();
for (size_t i = 0; i <s.size(); ++i) {
    s[i]->go(f);
}
```

- Add `void flag::init(const fastring& path);`



### Changed

- `Closure` to `co::Closure`.

- Improve `co::Event`, can be used anywhere, and support copy constructor and capture by value in lambda.

- Improve `co::Mutex`, `co::Pool`, support copy constructor and capture by value in lambda.

- `co::close()` now can be called anywhere, not necessary to call it in the thread that performed the IO operations.

- Partial support for mingw. Coroutine and coroutine-based features do not work for mingw at present.



### Bugs fixed

- fix bug in fs::file when read/write more than 4G bytes.

- fix connect timeout error for http::Client.

- fix link problem in #165.
