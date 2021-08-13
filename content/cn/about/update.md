---
weight: 4
title: "更新日志"
---


## v2.0.1


### 参考文档

- 中文: [github](https://idealvin.github.io/cn/about/co/) [gitee](https://idealvin.gitee.io/cn/about/co/)
- English: [github](https://idealvin.github.io/en/about/co/) [gitee](https://idealvin.gitee.io/en/about/co/)



### 新特性

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

- defer (类似于 golang 中的 defer)
```cpp
#include "co/defer.h"
Timer t;
defer(LOG << "time elapse: " << t.us() << "us");
```

- channel (类似于 golang 中的 channel)
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

- waitgroup (类似于 golang 中的 sync.WaitGroup)
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

- 协程在 windows 平台支持 hook。

- 在指定调度线程中创建协程。
```cpp
auto s = co::next_scheduler();
s->go(f1);
s->go(f2);
```

- 在所有调度线程中创建协程。
```cpp
auto& s = co::all_schedulers();
for (size_t i = 0; i <s.size(); ++i) {
    s[i]->go(f);
}
```

- 增加 `void flag::init(const fastring& path);`



### 改变

- 全局的 `Closure` 改为 `co::Closure`.

- 改进 `co::Event`, 可以在协程及非协程中使用, 支持拷贝构造、lambda 中按值捕获.

- 改进 `co::Mutex`, `co::Pool`, 支持拷贝构造、lambda 中按值捕获.

- 改进 `co::close()`, 可以在任何地方调用.

- 部分支持 mingw, 协程相关特性暂时不能在 mingw 上运行.



### 问题修复

- 修复 `fs::file` 读写超过 4G 长度数据的 bug.

- 修复 http::Client 连接超时时的错误信息.

- 修复 #165 中的链接问题.
