---
weight: 1
title: "一文了解 CO"
---


## CO 是什么

CO 是一个优雅、高效的 C++ 基础库，支持 Linux, Windows 与 Mac 等平台，它实现了类似 golang 的协程、基于协程的网络编程框架、命令行参数与配置文件解析库、高性能日志库、单元测试框架、JSON 库等一系列高质量的基础组件。

CO 在 [github](https://github.com/idealvin/co) 上以 [MIT](https://mit-license.org/) 许可证开源，它使用了部分三方代码，可能有不同的许可证，详情见 [LICENSE](https://github.com/idealvin/co/blob/master/LICENSE.md) 文件。为了方便国内用户，[gitee](https://gitee.com/idealvin/co) 上也会定期同步 github 上的代码。




## CO 的发展历程

[Alvin(idealvin)](https://github.com/idealvin) 自 2013 年开始开发 CO，最初的目的是为了减少 C++ 项目中的三方依赖，同时提高 C++ 的开发效率。从 2015 年开始，Alvin 将 CO 引入实际项目中，供自己与同事使用，大大缩减了项目的开发周期，CO 也得以经受工业项目的检验。

经过多年的积累、沉淀，到 2019 年，Alvin 又用 C++ 实现了 golang 中的协程机制，并提供了一套基于协程的网络编程框架。CO 协程诞生之初，就被用于嵌入式网络程序开发，并取得了立竿见影的效果。

截至 2021 年，CO 协程又有了长足的发展，目前在 Linux/Windows/Mac 平台均已支持 hook，并且实现了协程锁、协程同步事件、协程池以及 golang 中的 channel 与 waitgroup，用户可以用 CO 写出 golang 的体验。




## 快速上手

### 编译

建议安装 [xmake](https://github.com/xmake-io/xmake)，在 CO 根目录执行如下命令构建所有子项目：
```sh
xmake -a
```

如果需要使用 http::Client, SSL 或 HTTPS 特性，则可以用下面的命令构建：
```sh
xmake f --with_libcurl=true --with_openssl=true
xmake -a
```

xmake 会自动从网络安装 libcurl 与 openssl，视网络情况，这个过程可能会较慢。`xmake -a` 会构建 [libco](https://github.com/idealvin/co/tree/master/src), [gen](https://github.com/idealvin/co/tree/master/gen), [co/unitest](https://github.com/idealvin/co/tree/master/unitest) 以及 [co/test](https://github.com/idealvin/co/tree/master/test) 下面的所有测试代码。用户可以执行下面的命令，运行 CO 中的测试程序：

```sh
xmake r unitest -a
xmake r flag
xmake r log -cout
xmake r co
```



### 使用 CO 开发 C++ 项目

最简单的，可以直接包含 [co/all.h](https://github.com/idealvin/co/blob/master/include/co/all.h)，使用 CO 中的所有特性。如果担心影响编译速度，也可以只包含需要用到的头文件，如包含 [co/co.h](https://github.com/idealvin/co/blob/master/include/co/co.h)，可以使用 co/flag, co/log 以及协程相关的所有特性。


```cpp
#include "co/all.h"

DEF_string(s, "nice", "");

int main(int argc, char** argv) {
    flag::init(argc, argv);
    log::init();

    LOG << FLG_s;
    return 0;
}
```

上面是一个简单的例子，main 函数前两行分别用于初始化 flag 与 log 库。CO 中的部分组件会用 flag 定义配置项，用 log 打印日志，因此，一般需要在 main 函数开头调用 `flag::init()` 与 `log::init()` 进行初始化。

用户也可以用宏 `DEF_main` 定义 main 函数：

```cpp
#include "co/all.h"

DEF_string(s, "nice", "");

DEF_main(argc, argv) {
    LOG << FLG_s;
    return 0;
}
```

DEF_main 在内部已经调用了 `flag::init()` 与 `log::init()`，用户无需再次调用。另外，DEF_main 会将 main 函数中的代码放到协程中运行，与 golang 保持一致，golang 中的 main 函数也在协程中。CO 中部分协程相关的组件必须在协程中使用，用 CO 开发基于协程的应用程序时，一般建议用 DEF_main 定义 main 函数。




## 核心组件


### co/flag

[co/flag](../../co/flag/) 是一个简单易用的命令行参数与配置文件解析库，CO 中的一些组件会用它定义配置项。

co/flag 为每个配置项提供一个默认值，在没有配置参数的情况下，程序可以按默认配置运行。用户也可以从**命令行或配置文件**传入配置参数，在需要配置文件时，可以执行 `./exe -mkconf` **自动生成配置文件**。

```cpp
// xx.cc
#include "co/flag.h"
#include "co/log.h"

DEF_bool(x, false, "bool x");
DEF_bool(y, false, "bool y");
DEF_uint32(u32, 0, "...");
DEF_string(s, "hello world", "string");

int main(int argc, char** argv) {
    flag::init(argc, argv);

    COUT << "x: " << FLG_x;
    COUT << "y: " << FLG_y;
    COUT << "u32: " << FLG_u32;
    COUT << FLG_s << "|" << FLG_s.size();

    return 0;
}
```

上面是一个使用 co/flag 的例子，代码中 `DEF_` 开头的宏，定义了 4 个配置项，每个配置项相当于一个全局变量，变量名是 `FLG_` 加配置名。上面的代码编译完后，可以按下面的方式运行：

```sh
./xx                  # 按默认配置运行
./xx -xy -s good      # 单字母命名的 bool flag, 可以一并设置为 true
./xx -s "I'm ok"      # 含空格的字符串
./xx -u32 8k          # 整数可以带单位: k,m,g,t,p, 不区分大小写

./xx -mkconf          # 自动生成配置文件 xx.conf
./xx xx.conf          # 从配置文件传入参数
./xx -config xx.conf  # 与上同
```



### co/log

[co/log](../../co/log/) 是一个高性能的本地日志系统，CO 中的一些组件会用它打印日志。

co/log 将日志分为 debug, info, warning, error, fatal 5 个级别，**打印 fatal 级别的日志会终止程序的运行**。用户可以像下面这样打印不同级别的日志：

```cpp
DLOG << "hello " << 23;  // debug
LOG << "hello " << 23;   // info
WLOG << "hello " << 23;  // warning
ELOG << "hello " << 23;  // error
FLOG << "hello " << 23;  // fatal
```

co/log 还提供了一系列 `CHECK` 宏，可以视为加强版的 `assert`，它们在 debug 模式下也不会被清除。

```cpp
void* p = malloc(32);
CHECK(p != NULL) << "malloc failed..";
CHECK_NE(p, NULL) << "malloc failed..";
```

CHECK 断言失败时，co/log 会打印函数调用栈信息，然后终止程序的运行。

co/log 速度非常快，在程序运行稳定后，几乎不需要内存分配操作。下面是一些测试结果，仅供参考：

- co/log vs glog (single thread)

  | platform | google glog | co/log |
  | ------ | ------ | ------ |
  | win2012 HHD | 1.6MB/s | 180MB/s |
  | win10 SSD | 3.7MB/s | 560MB/s |
  | mac SSD | 17MB/s | 450MB/s |
  | linux SSD | 54MB/s | 1023MB/s |

- [co/log vs spdlog](https://github.com/idealvin/co/tree/benchmark) (Windows)

  | threads | total logs | co/log time(seconds) | spdlog time(seconds)|
  | ------ | ------ | ------ | ------ |
  | 1 | 1000000 | 0.103619 | 0.482525 |
  | 2 | 1000000 | 0.202246 | 0.565262 |
  | 4 | 1000000 | 0.330694 | 0.722709 |
  | 8 | 1000000 | 0.386760 | 1.322471 |

- [co/log vs spdlog](https://github.com/idealvin/co/tree/benchmark) (Linux)

  | threads | total logs | co/log time(seconds) | spdlog time(seconds)|
  | ------ | ------ | ------ | ------ |
  | 1 | 1000000 | 0.096445 | 2.006087 |
  | 2 | 1000000 | 0.142160 | 3.276006 |
  | 4 | 1000000 | 0.181407 | 4.339714 |
  | 8 | 1000000 | 0.303968 | 4.700860 |



### co/unitest

[co/unitest](../../co/unitest/) 是一个简单易用的单元测试框架，CO 中的很多组件会用它写单元测试代码，为 CO 的稳定性提供了保障。

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

上面是一个简单的例子，`DEF_test` 宏定义了一个测试单元，实际上就是一个函数(类中的方法)。`DEF_case` 宏定义了测试用例，每个测试用例实际上就是一个代码块。用户可以将不同的测试单元，放到同一个 C++ 项目中，main 函数一般只需要下面几行：

```cpp
#include "co/unitest.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);
    unitest::run_all_tests();
    return 0;
}
```

[co/unitest](https://github.com/idealvin/co/tree/master/unitest) 目录下面是 CO 中的单元测试代码，编译后可执行下述命令运行：

```sh
xmake r unitest -a   # 运行所有单元测试用例
xmake r unitest -os  # 仅运行 os 单元中的测试用例
```



### 协程

CO 实现了类似 [golang](https://github.com/golang/go) 的协程，它有如下特性：

- 支持多线程调度，默认线程数为系统 CPU 核数。
- 共享栈，同一线程中的协程共用若干个栈(大小默认为 1MB)，内存占用低，Linux 上的测试显示 1000 万协程只用了 2.8G 内存(仅供参考)。
- 各协程之间为平级关系，可以在任何地方(包括在协程中)创建新的协程。
- 支持系统 API hook (Windows/Linux/Mac)，可以在直接在协程中使用三方网络库。



#### 创建协程

```cpp
go(ku);            // void ku();
go(f, 7);          // void f(int);
go(&T::f, &o);     // void T::f(); T o;
go(&T::f, &o, 7);  // void T::f(int); T o;
go([](){
    LOG << "hello go";
});
```

上面是用 `go()` 创建协程的例子，go() 是一个函数，它接受 1 到 3 个参数，第一个参数 `f` 是任意可调用的对象，这些参数只要满足 `f()`, `(*f)()`, `f(p)`, `(*f)(p)`, `(o->*f)()` 或者 `(o->*f)(p)` 能被调用就可以了。

`go()` 创建的协程会均匀的分配到不同的调度线程中。如果用户想让某些协程运行在同一个线程下，可以用下面的方式创建协程：

```cpp
auto s = co::next_scheduler();
s->go(f1);
s->go(f2);
```

如果用户想在所有的调度线程中创建协程，可以用下面的方式：

```cpp
auto& s = co::all_schedulers();
for (size_t i = 0; i < s.size(); ++i) {
    s[i]->go(f);
}
```


#### channel

CO 实现了 `co::Chan`，类似于 golang 中的 channel，它是一个模板类，模板参数可以是内置类型、指针类型或者其他可以进行内存拷贝的结构体或类。

```cpp
#include "co/co.h"

DEF_main(argc, argv) {
    co::Chan<int> ch;
    go([ch]() {
        ch << 7;
    });

    int v = 0;
    ch >> v;
    LOG << "v: " << v;

    return 0;
}
```

上面是个简单的例子，需要注意，**co::Chan 必须在协程中使用**，因此代码中用 `DEF_main` 定义 main 函数，让 main 函数中的代码也运行在协程中。

另外，代码中的 channel 对象在栈上，而 CO 采用的是共享栈实现方式，一个协程栈上的数据可能被其他协程覆盖，**协程间一般不能直接共享栈上的数据**，因此代码中的 lambda 采用了**按值捕获**的方式，将 channel 拷贝了一份，传递到新建的协程中。channel 的拷贝操作只是将内部引用计数加 1，几乎不会对性能造成影响。

用户在创建 channel 时，还可以加上超时时间：

```cpp
co::Chan<int> ch(8, 1000);
go([ch]() {
    ch << 7;
    if (co::timeout()) LOG << "write to channel timeout";
});

int v = 0;
ch >> v;
if (!co::timeout()) LOG << "v: " << v;
```

co::Chan 构造函数的第一个参数是内部队列长度，第二个参数是超时时间，单位为毫秒。用户在 channel 读写操作后，可以用 `co::timeout()` 判断是否超时。


#### waitgroup



#### 协程