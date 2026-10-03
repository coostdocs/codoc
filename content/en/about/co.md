---
weight: 2
title: "Overview"
---

## Introduction

[coost](https://github.com/idealvin/coost) is a lightweight cross-platform C++ base library. It provides components such as coroutines, networking, RPC, logging, configuration, and JSON. Its style is close to golang, it pursues minimalism while maintaining high performance, and it does not depend on third-party libraries such as Boost or folly.

It is suitable for the following scenarios:
- You need lightweight coroutines;
- You need a coroutine-based network framework;
- You do not want to introduce heavy or numerous third-party libraries;
- You want configuration, logging, memory management, etc. to have a consistent style.

## Design Principles

coost's core design principle is keep simple, while maintaining high performance.

Specifically:
- Each component remains minimal and only provides commonly used functionality;
- Simple components combine to cover most application scenarios;
- While remaining simple, it pursues performance, balancing ease of use and performance.

Based on the above principles:
- coost usually provides short, semantically clear names;
- coost generally does not increase code complexity or maintenance difficulty just to support some rarely used features;
- coost does not use exceptions internally, so users do not need `try / catch`.

## What Problems It Solves

During C++ project development, the following problems are commonly encountered:
- **Complex concurrency model**: Multithreading + callbacks + locks make logic easy to get messy, and asynchronous code is hard to read and debug.
- **Many dependencies**: A project may introduce multiple third-party libraries with inconsistent code styles, memory management approaches, etc., resulting in high integration, debugging, and optimization costs.
- **Heavy dependencies**: A project may introduce large third-party libraries such as Boost or folly, leading to slow compilation and a high learning curve.
- **Chaotic global object management**: Static objects in different .cc files have undefined initialization and destruction order; the program may crash due to accessing uninitialized or already destroyed static objects.

## coost's Solutions

- Use coroutines to simplify the concurrency model and achieve asynchronous performance with synchronous code.
- Use coost as a single library to cover common base components such as configuration, logging, coroutines, networking, RPC, JSON, unit testing, and benchmarking, reducing third-party dependencies.
- Core components are self-developed and do not depend on large third-party libraries such as Boost or folly.
- Use `co::make_static / co::make_rootic` to take over static objects, and rely on the nifty counter technique to ensure that depended-upon objects are constructed first and destroyed later.

In addition, coost also unifies:
- **Configuration**: All components are configured through flags, supporting command line and configuration files.
- **Output**: Terminal output and logs are based on the `co::string` streaming interface; custom types only need to implement `operator<<(co::string&, const T&)`.
- **Memory allocation**: All components uniformly use the coost memory allocator.

## Core Components and Design Choices

### Coroutines

coost coroutines are similar to goroutines in golang:
- Use `go()` to create a coroutine;
- Support multi-threaded scheduling; the default number of scheduling threads is the number of system CPU cores;

Unlike C++20 stackless coroutines, coost coroutines do not have the "code pollution" problem: they do not require `co_await / co_yield / co_return`, do not require introducing types such as `task<T> / promise_type`, and do not require changing function signatures.

#### Shared Stack

coost coroutines use a shared stack design. **Coroutines in the same scheduler share a fixed number of stacks**, so the stack memory overhead per coroutine is extremely low, and a single machine can support tens of millions of concurrent coroutines.

Special attention is required: because a coroutine's stack is not exclusive, it is generally **not possible to access objects on a coroutine stack across coroutines through pointers or references**.

#### Parameter Passing in go

`go()` accepts parameters similar to the `std::thread` constructor; it can accept ordinary functions, member functions, lambdas, function objects, and any number and type of parameters.

The go function looks roughly like this: `go(f, args...)`. Passing args by reference is safe; however, **if f is a lambda, it generally cannot capture objects on the coroutine stack by reference**, because the coroutine stack is shared, and data on the stack may be overwritten by other coroutines.

#### Synchronization Mechanisms

coost coroutines provide three synchronization mechanisms: `co::mutex / co::event / co::wait_group`.

golang is a fully coroutine environment, while coost needs to handle both coroutine and non-coroutine environments. coost designs the above synchronization mechanisms to **support both coroutines and non-coroutines**. The implementation cost is higher, but it is convenient for users in mixed environments.

In addition, because of the shared stack, coost needs to follow the semantics of **passing parameters by value** in coroutines, just like golang. Therefore, the above synchronization mechanisms all use a reference-counting-based design, and copy operations only increase the reference count.

### Memory Allocator

coost has developed its own memory allocator, which is uniformly used by all components, making memory management, statistics, debugging, and optimization easier.

The allocator divides memory into three categories:
- Small memory: `< 4k`, 16-byte aligned;
- Medium memory: `<= 128k`, 4k-byte aligned;
- Large memory: `> 128k`, directly mmap / VirtualAlloc, page-aligned.

Unlike ordinary allocators, the coost memory allocator does not store the size of allocated memory; free requires passing the size. The benefits are:

- The allocator does not need to write metadata at the beginning of memory; the memory layout obtained by users is compact, with no extra overhead;
- During free, the level can be determined directly from the size without reading metadata, making the path simple;
- It is cache-friendly, and allocation and deallocation are faster.

#### Low-Cost Container Migration

`co/stl.h` provides aliases for **STL containers that use the coost memory allocator**. Users only need to replace `std::vector / std::map / std::unordered_map`, etc. with `co::vector / co::map / co::hash_map`, etc., to gain performance improvements without changing the code structure.

#### Static Object Management

Static objects in different .cc files in C++ have undefined initialization and destruction order; the program may crash due to accessing uninitialized or already destroyed static objects. coost's solution is:
- Do not directly define static objects; instead, define global pointers and let coost take over;
- Through the nifty counter technique, call `co::make_static<T>(args...)` to create a static object, and use its return value to initialize the global pointer.

The nifty counter naturally guarantees that depended-upon objects are constructed first. When the program exits, coost destroys the objects it has taken over in the order of first constructed, later destroyed.

In addition, coost also provides `co::make_rootic<T>(args...)`, used to **create static objects without dependencies**. coost guarantees that objects constructed by make_rootic are always destroyed last. make_rootic is generally used in scenarios where the nifty counter technique cannot be used, such as the initialization of dependency-free thread-local objects.

#### Writing Memory-Friendly Code

`co/def.h` defines `co::cache_line_size` and the `__cacheline_aligned` macro. When defining a struct, you can write `struct __cacheline_aligned S`, making the semantics clearer.

`co::alloc(n, align)` allocates memory with the specified alignment; align is at most 256 (the maximum possible value of co::cache_line_size). Users can use `co::alloc(n, co::cache_line_size)` to allocate cache-line-aligned memory.

### flag / log / unitest / benchmark

coost provides four base components: flag, log, unitest, and benchmark, corresponding to common scenarios of gflags, glog, gtest, and google benchmark, respectively.

- flag provides more powerful features, supports flag aliases and automatic configuration file generation, and integer flag values can carry units (k, m, g, t, p), making it more convenient to use.
- log has better performance than glog, usually improving by 1 to 2 orders of magnitude. Printing a large number of logs in business code may affect processing performance, so log places special emphasis on performance improvement in its design.
- unitest makes writing unit tests simpler. The test unit defined by `DEF_test` is actually a function, and `DEF_case` is just a code block within it. Users can freely add code in the function, such as initialization code shared by test cases.
- benchmark is designed similarly to unitest. The benchmark group defined by `BM_group` is also a function, and `BM_add` is just a code block within it, making it more convenient to use.

## A Minimal Example

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

This example demonstrates the basic usage of coost:
- `flag::parse` parses command-line arguments; without this line, the logging thread and coroutine scheduling threads will not start;
- `co::wait_group wg(2)` creates a wait group with a counter of 2;
- `go` starts two coroutines; the lambda captures `wg` by value, because coroutines use a shared stack and cannot capture objects on the coroutine stack by reference (although in this example wg is not on the coroutine stack, and capturing by reference would also work, for safety it is recommended to always capture by value);
- `co::println` immediately outputs to the terminal; `log::info` writes to a file by default and does not output to the terminal. To see log content in the terminal, add `-also_log2console=true` on the command line;
- `wg.wait()` waits on the main thread for the two coroutines to finish, then exits.

## Compilation and Running

coost supports gcc, clang, and MSVC compilers, and requires compiler support for C++17. coost can be built with xmake or cmake; xmake is recommended because it is more convenient to use.

Common commands (executed in the coost root directory):
```bash
# Build libco by default
xmake

# Build and run unit test code
xmake b unitest
xmake r unitest
xmake r unitest -os

# Build and run benchmark code
xmake b benchmark
xmake r benchmark
xmake r benchmark -mem

# Build and run test code under the test directory
xmake b xx
xmake r xx
```

Users can add their own `xxx.cc` files in the [test](https://github.com/idealvin/coost/tree/master/test) directory, and directly use `xmake b xxx` to build and `xmake r xxx` to run.

## Comparison with Similar Libraries

| Dimension | coost | Boost | folly |
| ---------- | ---------------- | ---------------- | ---------------- |
| Positioning | Lightweight base library collection | Large general-purpose library collection | Facebook internal base library collection |
| Coroutines | Yes, shared stack, non-C++20 coroutines | C++20 coroutines | Yes |
| Networking | TCP/UDP/RPC | Asio (TCP/UDP) | Multiple |
| RPC | Built-in | No | Yes |
| JSON | Built-in | Yes | Yes |
| flag / log | Built-in | No (requires gflags/glog) | Yes |
| Unit / Benchmark Testing | Built-in | Boost.Test | Yes |
| Memory Allocator | Self-developed | Standard | Self-developed |
| Configuration | Uniformly uses flag | Independent per component | Independent per component |
| Dependencies | None | Varies by component | Many |
| Learning Difficulty | Medium | High | High |
| Size | Small | Large | Large |

coost is not fully comparable to Boost or folly. coost is positioned as a "lightweight base library collection"; its coverage is smaller than Boost, but its components have a consistent style, it does not depend on third-party libraries, and its learning cost is lower. The table above compares only some dimensions and does not mean the functionality is completely equivalent.

## Concurrency Model

### Server Side

The server side usually needs to support high concurrency. coost adopts the basic model of "one connection, one coroutine":

- A single coroutine for accept is responsible for opening the listening socket, looping to accept, and closing the listening socket;
- Each time a new connection is accepted, a separate coroutine is started, responsible for recv / send / close on that connection.

The lifecycle of a connection is consistent with the lifecycle of a coroutine. Under this model, the processing logic for each connection is independent and sequential; synchronous code is sufficient, and no callbacks or manual state machine management are needed.

### Client Side

The client side needs to consider connection reuse, rather than establishing a new connection for every coroutine. coost provides `co::pool` to solve this problem:
- Inside `co::pool`, each scheduling thread has its own pool, and elements in the pool are not shared across threads, so using co::pool does not require locking;
- When a client coroutine needs one, it takes a connection from co::pool and immediately puts it back after use;

With the above model, a large number of client coroutines can usually share a small number of connections in `co::pool`, instead of creating one connection per coroutine.

With `co::pool`, the design of `co::tcp_client` and `co::rpc_client` in coost is very simple. A client allows only one coroutine to use it at the same time; users can put the client into co::pool to achieve reuse. In short, co::pool provides a general method for reusing client connections in a coroutine environment.

## Recommended Reading Order

1. flag: command-line and configuration file parsing;
2. log: logging;
3. print: terminal output;
4. string: strings;
5. mem: memory allocator;
6. co: coroutines;
7. sock: socket;
8. tcp: TCP;
9. json: JSON;
10. rpc: RPC;
11. gen: RPC code generation.
