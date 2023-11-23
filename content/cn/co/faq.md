---
weight: 33
title: "FAQ"
---

# FAQ

## 如何修改配置项的值？

**[flag](../flag/)** 是 coost 提供的命令行参数与配置文件解析库。coost 中的日志、协程、网络等模块使用 flag 定义了一些配置项，用户可以**通过命令行参数或配置文件修改配置项的值**。命令行中的用法见[命令行中使用 flag](../flag/#命令行中使用-flag)，配置文件的用法见[配置文件](../flag/#配置文件)。需要注意，**修改配置项需要在 main 函数开头调用 [flag::parse](../flag/#flagparse) 解析命令行参数**。

```cpp
int main(int argc, char** argv) {
    flag::parse(argc, argv);
    return 0;
}
```

用户也可以通过 API [flag::set_value](../flag/#flagset_value) 修改配置项的值：

```cpp
int main(int argc, char** argv) {
    flag::set_value("co_sched_num", "2");
    flag::set_value("version", "v3.1.4");
    flag::parse(argc, argv);
    return 0;
}
```

{{< hint info >}}
`flag::set_value` 在 `flag::parse` 之前，这样用户仍然能够通过命令行参数、配置文件修改相关配置项的值。
{{< /hint >}}



## 程序启动时如何指定配置文件？

用户使用 [flag](../flag/) 定义配置项后，程序启动时可用如下方式指定配置文件：

```cpp
./xx xx.conf          # 此种情况，配置文件名需要以 .conf 或 config 结尾
./xx -conf xx.conf    # 此种情况，配置文件名没有上述限制
```

注意 main 函数开头需要调用 [flag::parse](../flag/#flagparse) 方法。另外，用户**可以用 `-mkconf` 自动生成配置文件**：

```sh
# 自动生成配置文件 xx.conf
./xx -mkconf
```



## 如何使用 flag 别名？

[flag](../flag/) 支持别名，定义了别名的 flag，在命令行参数、配置文件中，可以用别名取代原名。flag 别名有两种用法，第一种是在定义 flag 时指定别名：


```cpp
DEF_bool(debug, false, "xxx");          // 无别名
DEF_bool(debug, false, "xxx", d);       // 别名 d
DEF_bool(debug, false, "xxx", d, dbg);  // 两个别名: d, dbg
```

还有一种情况，flag 已经定义，且用户无法修改 flag 的定义时，可以用 [flag::alias](../flag/#flagalias) 设置别名，如 coost 内部定义了 string 类型的 flag `version`，用户可以用如下方式给它设置别名：

```cpp
int main(int argc, char** argv) {
    flag::alias("version", "v");
    flag::parse(argc, argv);
    return 0;
}
```

{{< hint warning >}}
`flag::alias` 须在 `flag::parse` 前调用。
{{< /hint >}}



## 如何配置日志模块？

coost 日志模块使用 [flag](../flag/) 定义配置项，具体的配置项见 [log 参考文档](../log/#配置项)。用户可以在命令行参数或配置文件中修改这些配置项的值，具体用法见 [flag 参考文档](../flag/)。



## 如何设置 JSON 中的小数位数？

[co::Json](../json/) 的 [str](../json/jsonstr) 方法有个参数 `mdp`，可用于指定最大有效小数位数(默认是 16):

```cpp
co::Json x = {
    {"a", 3.14159},
    {"b", 1.2345e-5},
};
co::print(x.str());   // {"a":3.14159,"b":1.2345e-5}
co::print(x.str(2));  // {"a":3.14,"b":1.23e-5}
```



## 如何实现 JSON 与结构体互转？

coost v3.0.1 基于 flex 与 [byacc](https://invisible-island.net/byacc/) 重写了代码生成工具 [gen](https://github.com/idealvin/coost/tree/master/gen)，可以根据 proto 文件中结构体的定义，自动生成 JSON 与结构体互转的代码，代码示例见 [j2s](https://github.com/idealvin/coost/tree/master/test/j2s)。




## 频繁创建、销毁线程导致内存泄露？

**频繁创建、销毁线程是设计上的错误。**

coost 中使用了 TLS (线程局部存储)，频繁创建、销毁线程时，一些 TLS 资源在线程退出时并不会自动销毁，从而导致内存泄露。

尽管 C++ 标准中的 `thread_local`，可以保证 TLS 对象在线程退出时自动析构，但是**在线程退出时析构 TLS 对象并不总是正确的行为**，如基于 TLS 实现的内存分配器，一个线程退出时，该线程中分配的内存，可能还在被其他线程使用，这个时候释放该线程的 TLS 资源，可能会导致程序崩溃。



## 如何设置协程调度线程的数量？

coost 中协程支持多线程调度，默认线程数为系统 CPU 核数，用户可以通过配置项(flag) `co_sched_num` 设置该值，**该值最大不能超过系统 CPU 核数**。配置用法可参考[如何修改配置项的值](#如何修改配置项的值)。



## 创建协程的数量是否有限制？

创建协程的数量仅受限于系统物理资源(主要是内存)。coost 协程采用共用栈的实现方式，实际内存占用量较少。Linux 上的简单测试显示 1000 万协程只用了 2.8G 内存(仅供参考)。



## 协程是否会出现栈溢出？

coost 协程默认栈大小为 `1M`，足够大，一般不会出现栈溢出。如果 1M 不够，用户可[通过配置项改大一些](../concurrency/coroutine/conf/#co_stack_size)。



## 协程共享栈在使用上有哪些限制？

**协程中不要直接访问其他协程中的局部变量**，因为共享栈上的局部变量可能被覆盖。



## 协程中可以使用三方网络库吗？

coost 协程 hook 了系统 socket 相关 API，可以在协程中直接使用 mysql、redis 等三方网络库。详情见[协程中使用三方网络库](../net/third/)。



## 如何将主线程变成协程调度线程？

coost 中调度线程默认是独立于主线程运行的，可借助 [co::main_sched](../concurrency/coroutine/api/#comain_sched) 将主线程设置为协程调度线程：

```cpp
#include "co/co.h"
#include "co/cout.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::print("main thread id: ", co::thread_id());

    auto s = co::main_sched();
    for (int i = 0; i < 8; ++i) {
        go([]{
            co::print("thread: ", co::thread_id(), " sched: ", co::sched_id());
        });
    }
    s->loop(); // loop forever
    return 0;  // fake return
}
```

{{< hint warning >}}
须在**创建协程之前**调用 `co::main_sched` (该方法返回指向 `MainSched` 的指针) 将主线程标记为调度线程，创建协程后，再调用 `MainSched` 的 `loop` 方法启动该协程调度器。
{{< /hint >}}
