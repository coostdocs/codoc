---
weight: 1
title: "Introduction"
---

## What is coost

[![stars](https://img.shields.io/github/stars/idealvin/coost?style=social)](https://github.com/idealvin/coost)
[![forks](https://img.shields.io/github/forks/idealvin/coost?style=social)](https://github.com/idealvin/coost)
[![MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)


**[coost](https://github.com/idealvin/coost)** is an elegant and efficient cross-platform C++ base library. Its goal is to create a sword of C++ to make C++ programming easy and enjoyable.

The original name of coost is **co** or cocoyaxi. It is like [boost](https://www.boost.org/), but more lightweight, **the static library built on linux or mac is only about 1MB in size**. However, it still provides enough powerful features:

<table>
<tr><td width=33% valign=top>

- Command line and config file parser (flag)
- **High performance log library (log)**
- Unit testing framework
- **go-style coroutine**
- Coroutine-based network library
- Efficient JSON library
- **JSON RPC framework**

</td><td width=34% valign=top>

- Atomic operation (atomic)
- **Efficient stream (fastream)**
- Efficient string (fastring)
- String utility (str)
- Time library (time)
- Thread library (thread)
- Timed Task Scheduler

</td><td valign=top>

- **God-oriented programming**
- LruMap
- hash library
- path library
- File utilities (fs)
- System operations (os)
- **Fast memory allocator**
 
</td></tr>
</table>




## History of coost

- **2013-2015**, [Alvin(idealvin)](https://github.com/idealvin) felt a little cumbersome when using google's gflags, glog, gtest, etc., so he implemented the corresponding functions by himself, that is, the **flag, log, unitest, etc.** in today's coost.

- **2015-2018**, Alvin introduced this library into actual projects for use by himself and his colleagues, which greatly improved the efficiency of C++ development. It has also been tested by industrial projects, and continuously improved and expanded new features in practice.

- **In 2019**, Alvin implemented a **go-style coroutine** and a network programming framework based on coroutines, and then **named the project co and release v1.0 on [github](https://github.com/idealvin/coost)**.

- **2020-2021**, improve hook mechanism, coroutine synchronization mechanism, add channel, defer and other features in golang, **release 2.x version**. During this period, **some githubers provided a lot of valuable suggestions, and helped to improve [xmake](https://github.com/xmake-io/xmake), cmake building scripts and many features in coost**.

- **2022**, add **a fast memory allocator**, improve overall performance, make major improvements to many components such as flag, log, JSON, RPC, fastring, fastream, and **rename the project coost, release version 3.0**.




## Quick Start


### Compiling

It is recommended to install [xmake](https://github.com/xmake-io/xmake) and run the following command in the root directory of coost to build all sub-projects:

```sh
xmake -a
```

If you need to use http::Client, SSL or HTTPS features, you can use the following command to build:

```sh
xmake f --with_libcurl=true --with_openssl=true
xmake -a
```

Xmake will automatically install libcurl and openssl from the network. Depending on the network, this process may be slow. `xmake -a` will build [libco](https://github.com/idealvin/coost/tree/master/src), [gen](https://github.com/idealvin/coost/tree/master/gen), [co/unitest](https://github.com/idealvin/coost/tree/master/unitest) and [co/test](https://github.com/idealvin/coost/tree/master/test). Users can run test programs in coost with the following commands:

```sh
xmake r unitest
xmake r flag
xmake r log -cout
xmake r co
```



### Develop C++ programs with coost

The simplest, you can directly include [co/all.h](https://github.com/idealvin/coost/blob/master/include/co/all.h) and use all the features in coost. If you are worried about the compiling speed, you can also include only the header files that you need, such as including [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h), you can use co/flag, co/log and all features related to coroutines.

```cpp
#include "co/all.h"

DEF_string(s, "nice", "");

int main(int argc, char** argv) {
    flag::init(argc, argv);
    LOG << FLG_s;
    return 0;
}
```

The above is a simple example. The first line of the main function is used to parse the command-line flags and the config file. Some components in coost use co/flag to define config items. Therefore, it is generally necessary to call `flag::init()` at the beginning of the main function for initialization.

Users can also use the macro `DEF_main` to define the main function:

```cpp
#include "co/all.h"

DEF_string(s, "nice", "");

DEF_main(argc, argv) {
    LOG << FLG_s;
    return 0;
}
```

`DEF_main` puts code in the main function into a coroutine, and `flag::init()` has been called internally, and users needn't call it manually.




## Performance

### Memory allocator

For memory allocators such as ptmalloc, jemalloc, tcmalloc and mimalloc, there is a high probability that the small memory will not be returned to the operating system after they are freed. To solve this problem, coost has designed a dedicated memory allocator (co/malloc), which will return as much released memory to the system as possible while taking into account performance, which is conducive to reducing the memory footprint of the program.

[co/test](https://github.com/idealvin/coost/blob/master/test/mem.cc) provides a simple test code, which can be built and run as follow:

````sh
xmake b mem
xmake r mem -t 4 -s
````

`-t` specifies the number of threads, `-s` means to compare with the system memory allocator. Here are the test results on different platforms (4 threads):

| os/cpu | co::alloc | co::free | ::malloc | ::free | speedup |
| ------ | ------ | ------ | ------ | ------ | ------ |
| win/AMD 3.2G | 7.32 | 6.83 | 86.05 | 105.06 | 11.7/15.3 |
| mac/i7 2.4G | 9.91 | 9.86 | 55.64 | 60.20 | 5.6/6.1 |
| linux/i7 2.2G | 10.80 | 7.51 | 1070.5 | 21.17 | 99.1/2.8 |

The data in the table is the average time, the unit is nanoseconds (ns), linux is the ubuntu system running in Windows WSL, speedup is the performance improvement multiple of coost memory allocator relative to the system memory allocator.

It can be seen that **co::alloc is nearly 99 times faster than ::malloc** on Linux. One of the reasons is that ptmalloc has a large lock competition overhead in multi-threaded environment, and co/malloc is designed to avoid the use of locks as much as possible. The allocation and release of small blocks of memory do not require locks, and even spin locks are not used when releasing across threads.



### Log library

| platform | glog | co/log | speedup |
| ------ | ------ | ------ | ------ |
| win2012 HHD | 1.6MB/s | 180MB/s | 112.5 |
| win10 SSD | 3.7MB/s | 560MB/s | 151.3 |
| mac SSD | 17MB/s | 450MB/s | 26.4 |
| linux SSD | 54MB/s | 1023MB/s | 18.9 |

The above is the write speed of co/log and glog (single thread, 1 million logs). It can be seen that co/log is nearly two orders of magnitude faster than glog.

| threads | linux co/log | linux spdlog | win co/log | win spdlog | speedup |
| ------ | ------ | ------ | ------ | ------ | ------ |
| 1 | 0.087235 | 2.076172 | 0.117704 | 0.461156 | 23.8/3.9 |
| 2 | 0.183160 | 3.729386 | 0.158122 | 0.511769 | 20.3/3.2 |
| 4 | 0.206712 | 4.764238 | 0.316607 | 0.743227 | 23.0/2.3 |
| 8 | 0.302088 | 3.963644 | 0.406025 | 1.417387 | 13.1/3.5 |

The above is the time of [printing 1 million logs with 1, 2, 4, and 8 threads](https://github.com/idealvin/coost/tree/benchmark), in seconds. Speedup is the performance improvement of co/log compared to spdlog on linux and windows platforms.



### JSON library

| os | co/json stringify | co/json parse | rapidjson stringify | rapidjson parse | speedup |
| ------ | ------ | ------ | ------ | ------ | ------ |
| win | 569 | 924 | 2089 | 2495 | 3.6/2.7 |
| mac | 783 | 1097 | 1289 | 1658 | 1.6/1.5 |
| linux | 468 | 764 | 1359 | 1070 | 2.9/1.4 |

The above is the average time of stringifying and parsing minimized [twitter.json](https://raw.githubusercontent.com/simdjson/simdjson/master/jsonexamples/twitter.json), in microseconds (us), speedup is the performance improvement of co/json compared to rapidjson.




## Core features


### God-oriented programming

[co/god.h](https://github.com/idealvin/coost/blob/master/include/co/god.h) provides some features based on templates.

```cpp
#include "co/god.h"

void f() {
    god::bless_no_bugs();
    god::align_up<8>(31); // -> 32
    god::is_same<T, int, bool>(); // T is int or bool?
}
```



### flag

**[flag](../../co/flag/)** is a simple and easy-to-use command line and config file parsing library. Some components in coost use it to define config items.

Each **flag(config item)** has a default value, and by default, the program can run with the default config values. Users can also pass in parameters from the **command line or config file**, and when a config file is required, `-mkconf` can be used to **generate it automatically**.

```cpp
#include "co/flag.h"
#include "co/cout.h"

DEF_bool(x, false, "x");
DEF_bool(debug, false, "dbg", d);
DEF_uint32(u, 0, "xxx");
DEF_string(s, "", "xx");

int main(int argc, char** argv) {
    flag::init(argc, argv);
    COUT << "x: " << FLG_x;
    COUT << "y: " << FLG_y;
    COUT << "debug: " << FLG_debug;
    COUT << "u: " << FLG_u;
    COUT << FLG_s << "|" << FLG_s.size();
    return 0;
}
```

In the above example, the macros start with `DEF_` define 4 flags. Each flag corresponds to a global variable, whose name is `FLG_` plus the flag name. The flag `debug` has an alias `d`. After building, the above code can run as follow:

```sh
./xx                  # Run with default configs
./xx -x -s good       # x -> true, s -> "good"
./xx -debug           # debug -> true
./xx -xd              # x -> true, debug -> true
./xx -u 8k            # u -> 8192

./xx -mkconf          # Automatically generate a config file: xx.conf
./xx xx.conf          # run with a config file
./xx -conf xx.conf    # Same as above
```



### log

**[log](../../co/log/)** is a high-performance log library, some components in coost use it to print logs.

log supports two types of logs: one is level log, which is divided into 5 levels: debug, info, warning, error and fatal, **printing a fatal log will terminate the program**; the other is topic log, logs are grouped by topic, and logs of different topics are written to different files.

```cpp
#include "co/log.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);

    TLOG("xx") << "s" << 23; // topic log
    DLOG << "hello " << 23;  // debug
    LOG << "hello " << 23;   // info
    WLOG << "hello " << 23;  // warning
    ELOG << "hello " << 23;  // error
    FLOG << "hello " << 23;  // fatal

    return 0;
}
```

co/log also provides a series of `CHECK` macros, which is an enhanced version of `assert`, and they will not be cleared in debug mode.

```cpp
void* p = malloc(32);
CHECK(p != NULL) << "malloc failed..";
CHECK_NE(p, NULL) << "malloc failed..";
```

log is very fast, the following are some test results:

| platform | glog | co/log | speedup |
| ------ | ------ | ------ | ------ |
| win2012 HHD | 1.6MB/s | 180MB/s | 112.5 |
| win10 SSD | 3.7MB/s | 560MB/s | 151.3 |
| mac SSD | 17MB/s | 450MB/s | 26.4 |
| linux SSD | 54MB/s | 1023MB/s | 18.9 |

The above is the write speed of co/log and glog (single thread, 1 million logs). It can be seen that co/log is nearly two orders of magnitude faster than glog.

| threads | linux co/log | linux spdlog | win co/log | win spdlog | speedup |
| ------ | ------ | ------ | ------ | ------ | ------ |
| 1 | 0.087235 | 2.076172 | 0.117704 | 0.461156 | 23.8/3.9 |
| 2 | 0.183160 | 3.729386 | 0.158122 | 0.511769 | 20.3/3.2 |
| 4 | 0.206712 | 4.764238 | 0.316607 | 0.743227 | 23.0/2.3 |
| 8 | 0.302088 | 3.963644 | 0.406025 | 1.417387 | 13.1/3.5 |

The above is the time of [printing 1 million logs with 1, 2, 4, and 8 threads](https://github.com/idealvin/coost/tree/benchmark), in seconds. Speedup is the performance improvement of co/log compared to spdlog on linux and windows platforms.



### unitest

**[unitest](../../co/unitest/)** is a simple and easy-to-use unit test framework. Many components in coost use it to write unit test code, which guarantees the stability of coost.

```cpp
#include "co/unitest.h"
#include "co/os.h"

namespace test {
    
DEF_test(os) {
    DEF_case(homedir) {
        EXPECT_NE(os::homedir(), "");
    }

    DEF_case(cpunum) {
        EXPECT_GT(os::cpunum(), 0);
    }
}
    
} // namespace test
```

The above is a simple example. The `DEF_test` macro defines a test unit, which is actually a function (a method in a class). The `DEF_case` macro defines test cases, and each test case is actually a code block. The main function is simple as below:

```cpp
#include "co/unitest.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);
    unitest::run_all_tests();
    return 0;
}
```

The directory [unitest](https://github.com/idealvin/coost/tree/master/unitest) contains the unit test code in coost. Users can run unitest with the following commands:

```sh
xmake r unitest -a   # Run all test cases
xmake r unitest -os  # Run test cases in the os unit
```



### JSON

In coost v3.0, **[Json](https://github.com/idealvin/coost/blob/master/include/co/json.h)** provides **fluent APIs**, which is more convenient to use.

```cpp
// {"a":23,"b":false,"s":"123","v":[1,2,3],"o":{"xx":0}}
Json x = {
    { "a", 23 },
    { "b", false },
    { "s", "123" },
    { "v", {1,2,3} },
    { "o", {
        {"xx", 0}
    }},
};

// equal to x
Json y = Json()
    .add_member("a", 23)
    .add_member("b", false)
    .add_member("s", "123")
    .add_member("v", Json().push_back(1).push_back(2).push_back(3))
    .add_member("o", Json().add_member("xx", 0));

x.get("a").as_int();       // 23
x.get("s").as_string();    // "123"
x.get("s").as_int();       // 123, string -> int
x.get("v", 0).as_int();    // 1
x.get("v", 2).as_int();    // 3
x.get("o", "xx").as_int(); // 0

x["a"] == 23;          // true
x["s"] == "123";       // true
x.get("o", "xx") != 0; // false
```



### Coroutine

coost has implemented a [go-style](https://github.com/golang/go) coroutine, which has the following features:

- Support multi-thread scheduling, the default number of threads is the number of system CPU cores.
- Shared stack, coroutines in the same thread share several stacks (the default size is 1MB), and the memory usage is low.
- There is a flat relationship between coroutines, and new coroutines can be created from anywhere (including in coroutines).
- Support coroutine synchronization events, coroutine locks, channels, and waitgroups.

```cpp
#include "co/co.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);

    co::WaitGroup wg;
    wg.add(2);

    go([wg](){
        LOG << "hello world";
        wg.done();
    });

    go([wg](){
        LOG << "hello again";
        wg.done();
    });

    wg.wait();
    return 0;
}
```

In the above code, the coroutines created by `go()` will be evenly distributed to different scheduling threads. Users can also control the scheduling of coroutines by themselves:

```cpp
// run f1 and f2 in the same scheduler
auto s = co::next_scheduler();
s->go(f1);
s->go(f2);

// run f in all schedulers
for (auto& s : co::schedulers()) {
    s->go(f);
}
```



### network programming

coost provides a coroutine-based network programming framework, which can be roughly divided into 3 parts:

- **[coroutineized socket API](../../co/coroutine/#coroutineized-socket-api)**, similar in form to the system socket API, users familiar with socket programming can easily write high-performance network programs in a synchronous manner.
- [TCP](../../co/net/tcp/), [HTTP](../../co/net/http/), [RPC](../../co/net/rpc/) and other high-level network programming components, compatible with IPv6, also support SSL, it is more convenient to use than socket API.
- **[System API hook](../../co/coroutine/#system-api-hook)**, with which, third-party network libraries can be used directly in coroutines.


**RPC server**

```cpp
#include "co/co.h"
#include "co/rpc.h"
#include "co/time.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);

    rpc::Server()
        .add_service(new xx::HelloWorldImpl)
        .start("127.0.0.1", 7788, "/xx");

    for (;;) sleep::sec(80000);
    return 0;
}
```

`rpc::Server` also supports HTTP protocol, you may use the POST method to call the RPC service:

```sh
curl http://127.0.0.1:7788/xx --request POST --data '{"api":"ping"}'
```


**Static web server**

```cpp
#include "co/flag.h"
#include "co/http.h"

DEF_string(d, ".", "root dir"); // docroot for the web server

int main(int argc, char** argv) {
    flag::init(argc, argv);
    so::easy(FLG_d.c_str()); // mum never have to worry again
    return 0;
}
```


**HTTP server**

```cpp
void cb(const http::Req& req, http::Res& res) {
    if (req.is_method_get()) {
        if (req.url() == "/hello") {
            res.set_status(200);
            res.set_body("hello world");
        } else {
            res.set_status(404);
        }
    } else {
        res.set_status(405); // method not allowed
    }
}

// http
http::Server().on_req(cb).start("0.0.0.0", 80);

// https
http::Server().on_req(cb).start(
    "0.0.0.0", 443, "privkey.pem", "certificate.pem"
);
```


**HTTP client**

```cpp
void f() {
    http::Client c("https://github.com");

    c.get("/");
    LOG << "response code: "<< c.status();
    LOG << "body size: "<< c.body().size();
    LOG << "Content-Length: "<< c.header("Content-Length");
    LOG << c.header();

    c.post("/hello", "data xxx");
    LOG << "response code: "<< c.status();
}

go(f);
```

