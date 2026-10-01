---
weight: 2
title: "简介"
---


## 定位

[coost](https://github.com/idealvin/coost) 是一个轻量的跨平台 C++ 基础库，提供协程、网络、RPC、日志、配置、JSON 等组件，风格接近 golang，追求极简，同时保持高性能，不依赖 Boost、folly 等三方库。

适合如下场景：
- 需要轻量协程；
- 需要基于协程的网络框架；
- 不想引入较重或较多的三方库；
- 希望配置、日志、内存管理等有一致的风格。


## 设计原则

coost 的核心设计原则是 keep simple，同时保持高性能。

具体来说：
- 单个组件保持极简，只提供常用功能；
- 简单组件组合起来，覆盖大部分应用场景；
- 在保持简单的同时追求性能，兼顾易用性与性能。

基于上述原则：
- coost 通常会提供简短、语义清晰的命名；
- coost 一般不会为了支持一些较少使用的功能，而增加代码的复杂度、维护难度；
- coost 内部不使用异常，用户不需要 `try / catch`。


## 它解决什么问题

C++ 项目开发中，通常会遇到下面问题：
- **并发模型复杂**：多线程 + 回调 + 锁，逻辑容易写乱，异步代码可读性差、难以调试。
- **依赖多**：项目可能引入多个三方库，代码风格、内存管理方式等不统一，集成、调试和优化成本高。
- **依赖重**：项目可能引入 Boost、folly 等大型三方库，编译慢、上手成本高。
- **全局对象管理混乱**：不同 .cc 文件的静态对象，初始化与析构顺序未定义，程序可能因访问未初始化或已析构的静态对象而崩溃。


## coost 的解决方案

- 用协程简化并发模型，用同步写法获得异步性能。
- 用 coost 一个库覆盖配置、日志、协程、网络、RPC、JSON、单元测试、基准测试等常用基础组件，减少三方依赖。
- 核心组件自研，不依赖 Boost、folly 等大型三方库。
- 用 `co::make_static / co::make_rootic` 接管静态对象，借助 nifty counter 技巧保证被依赖的对象先构造、后析构。

此外，coost 还统一了：
- **配置方式**：所有组件通过 flag 配置，支持命令行和配置文件。
- **输出方式**：终端输出、日志都基于 `co::string` 流式接口，自定义类型只需实现 `operator<<(co::string&, const T&)`。
- **内存分配**：所有组件统一使用 coost 内存分配器。


## 核心组件与设计选择

### 协程

coost 协程与 golang 中 goroutine 类似：
- 使用 `go()` 创建协程；
- 支持多线程调度，默认调度线程数为系统 CPU 核数；

区别于 C++20 无栈协程，coost 协程没有「代码污染」问题：不需要 `co_await / co_yield / co_return`，不需要引入 `task<T> / promise_type`等类型，不需要改函数签名。

#### 共享栈

coost 协程采用共享栈设计，**同一调度器中的协程共享固定数量的栈**，每个协程的栈内存开销极低，单机支持千万级协程并发。

需要特别注意，由于协程的栈不是独占的，通常**不能通过指针、引用跨协程访问协程栈上的对象**。

#### go 的参数传递

`go()` 接受的参数与 `std::thread` 构造函数类似，可以是普通函数、成员函数、lambda、函数对象，以及任意数量和类型的参数。

go 函数大致长这样：`go(f, args...)`，args 传引用是安全的；但**如果 f 是 lambda，则一般不能按引用捕获协程栈上的对象**，因为协程的栈是共享的，栈上数据可能被其他协程覆盖。


#### 同步机制

coost 协程提供 `co::mutex / co::event / co::wait_group` 三种同步机制。

golang 是全协程环境，而 coost 需要同时面对协程与非协程环境。coost 将上述同步机制设计为：**既支持协程，也支持非协程**。实现成本更高，但方便用户在混合环境下使用。

另外，由于是共享栈，coost 需要像 golang 一样，在协程中贯彻**按值传递参数**的语义。因此上述同步机制均采用基于引用计数的设计，拷贝操作只会增加引用计数。


### 内存分配器

coost 自研了一套内存分配器，所有组件统一使用，便于内存管理 、统计、调试和优化。

分配器将内存分为三类：
- 小内存：`< 4k`，16 字节对齐；
- 中等内存：`<= 128k`，4k 字节对齐；
- 大内存：`> 128k`，直接 mmap / VirtualAlloc，页对齐。

与一般分配器不同，coost 内存分配器不保存所分配内存的大小，free 时需要传大小。这样做的好处是：

- 分配器不需要在内存头部写元数据，用户拿到的内存布局紧凑，没有额外开销；
- free 时根据大小直接判断属于哪一级，不需要读元数据，路径简单；
- 缓存友好，分配和释放更快。

#### 容器低成本迁移

`co/stl.h` 提供**使用 coost 内存分配器的 STL 容器**别名。用户只需要将 `std::vector / std::map / std::unordered_map` 等替换成 `co::vector / co::map / co::hash_map` 等，就能带来性能上的提升，不需要修改代码结构。

#### 静态对象管理

C++ 不同 .cc 文件中的静态对象，初始化与析构顺序未定义，程序可能因访问未初始化或已析构的静态对象而崩溃。coost 给出的解决方案是：
- 不直接定义静态对象，而是定义全局指针，由 coost 接管；
- 通过 nifty counter 技巧，调用 `co::make_static<T>(args...)` 创建静态对象，并用其返回值初始化全局指针。

nifty counter 天然保证被依赖的对象先构造完成，coost 则在程序退出时，按照先构造、后析构的顺序，析构接管的对象。

另外，coost 也提供 `co::make_rootic<T>(args...)`，用于**创建无依赖的静态对象**，coost 保证 make_rootic 构建的对象总是在最后析构。make_rootic 一般用于无法使用 nifty counter 技巧的场景，如无依赖的线程局部对象的初始化。

#### 写内存友好的代码

`co/def.h` 定义了 `co::cache_line_size`，以及 `__cacheline_aligned` 宏。定义结构体时可以写 `struct __cacheline_aligned S`，语义更明确。

`co::alloc(n, align)` 分配指定对齐的内存，align 最大是 256(co::cache_line_size 的最大可能值)。用户可以用 `co::alloc(n, co::cache_line_size)` 分配缓存行对齐的内存。


### flag / log / unitest / benchmark

coost 提供 flag、log、unitest、benchmark 四个基础组件，分别对应 gflags、glog、gtest、google benchmark 的常用场景。

- flag 提供更强的功能，支持 flag 别名、自动生成配置文件，整型 flag 值可以带单位(k,m,g,t,p)，用起来更方便。
- log 性能比 glog 更好，通常有 1 到 2 个数量级的提升。业务代码中打印大量日志可能影响处理性能，所以 log 在设计上特别重视性能的提升。
- unitest 写单元测试更简单，`DEF_test` 定义的测试单元实际是一个函数，`DEF_case` 只是其中的代码块，用户可以在函数中自由添加代码，如各测试用例共用的初始化代码。
- benchmark 设计上与 unitest 类似，`BM_group` 定义的基准测试组也是一个函数，`BM_add` 只是其中的代码块，用起来更方便。


## 一个最小示例

```cpp
#include "co/co.h"
#include "co/flag.h"
#include "co/log.h"
#include "co/print.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    co::wait_group wg(2);

    go([wg](){
        co::println("hello world");
        wg.done();
    });

    go([wg](){
        log::info("hello again");
        wg.done();
    });

    wg.wait();
    return 0;
}
```

这个示例展示了 coost 的基本用法：
- `flag::parse` 解析命令行参数，没有这一行，日志线程、协程调度线程不会启动；
- `co::wait_group wg(2)` 创建一个计数器为 2 的等待组；
- `go` 启动两个协程，lambda 按值捕获 `wg`，因为协程是共享栈，不能按引用捕获协程栈上的对象(虽然示例中 wg 并不在协程栈上，按引用捕获也是可以的，但为安全起见，建议统一按值捕获)；
- `co::println` 立即输出到终端，`log::info` 默认写文件，不输出终端，想在终端看日志内容可以在命令行加 `-also_log2console=true`；
- `wg.wait()` 在主线程等待两个协程执行完，然后退出。


## 编译与运行

coost 支持的编译器有 gcc、clang、MSVC，需要编译器支持 C++17。coost 支持用 xmake 或 cmake 构建，推荐用 xmake，用起来更方便。

常用命令（在 coost 根目录下执行）：
```bash
# 默认编译 libco
xmake

# 编译及运行单元测试代码
xmake b unitest
xmake r unitest
xmake r unitest -os

# 编译及运行性能测试代码
xmake b benchmark
xmake r benchmark
xmake r benchmark -mem

# 编译及运行 test 目录下的测试代码
xmake b xx
xmake r xx
```

用户可以在 [test](https://github.com/idealvin/coost/tree/master/test) 目录添加自己的 `xxx.cc` 文件，直接 `xmake b xxx` 构建、`xmake r xxx` 运行。


## 和同类库的比较

| 维度         | coost            | Boost            | folly            |
| ---------- | ---------------- | ---------------- | ---------------- |
| 定位         | 轻量级基础库集合         | 大型通用库集合          | Facebook 内部基础库集合 |
| 协程         | 有，共享栈，非 C++20 协程 | C++20 协程         | 有                |
| 网络         | TCP/UDP/RPC      | Asio（TCP/UDP）    | 多种               |
| RPC        | 内置               | 无                | 有                |
| JSON       | 内置               | 有                | 有                |
| flag / log | 内置               | 无（需 gflags/glog） | 有                |
| 单元 / 基准测试  | 内置               | Boost.Test       | 有                |
| 内存分配器      | 自研               | 标准               | 自研               |
| 配置         | 统一使用 flag        | 各组件独立            | 各组件独立            |
| 依赖         | 无                | 各组件依赖不同          | 大量               |
| 上手难度       | 中                | 高                | 高                |
| 体量         | 小                | 大                | 大                |

coost 并不完全对标 Boost 或 folly。coost 的定位是「轻量级基础库集合」，覆盖面比 Boost 小，但组件之间风格统一，不依赖第三方库，上手成本更低。上表只从部分维度做对照，不代表功能完全等价。


## 并发模型

### 服务端

服务端通常需要支持高并发。coost 采用「一个连接一个协程」的基本模型：

- accept 单独一个协程，负责打开监听 socket、循环 accept、关闭监听 socket；
- 每 accept 一个新连接，单独开一个协程，负责该连接上的 recv / send / close。

连接的生命周期与协程的生命周期一致。这种模型下，每个连接的处理逻辑是独立的、顺序的，用同步的写法即可，不需要回调，也不需要手动管理状态机。


### 客户端

客户端需要考虑的是连接复用，而不是每个协程都建立新的连接。coost 提供 `co::pool` 解决这个问题：
- `co::pool` 内部每个调度线程都有自己的池子，池中的元素不会跨线程共享，因此使用 co::pool 不需要加锁；
- 客户端协程需要的时候，从 co::pool 中取出一个连接，用完立即放回；

使用上述模型，大量的客户端协程，通常可以共用 `co::pool` 中的少量连接，而不用为每个协程创建一个连接。

有了 `co::pool`，coost 中 `co::tcp_client`、`co::rpc_client` 的设计就很简单，一个 client 在同一时刻只允许一个协程使用，用户可以将 client 放入 co::pool，达到复用的目的。简言之，co::pool 提供了在协程环境复用客户端连接的通用方法。


## 推荐阅读顺序

1. flag：命令行与配置文件解析；
2. log：日志；
3. print：终端输出；
4. string：字符串；
5. mem：内存分配器；
6. co：协程；
7. sock：socket；
8. tcp：TCP；
9. json：JSON；
10. rpc：RPC；
11. gen：RPC 代码生成。
