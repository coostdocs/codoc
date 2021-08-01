---
weight: 11
title: "协程"
---

include: [co/co.h](https://github.com/idealvin/co/blob/master/include/co/co.h).


## 基本概念


- 协程是运行于线程中的轻量级调度单位。
- 协程之于线程，类似于线程之于进程。
- 一个进程中可以存在多个线程，一个线程中可以存在多个协程。
- 协程所在的线程一般被称为**调度线程**。
- 协程发生 io 阻塞或调用 sleep 等操作时，调度线程会挂起此协程。
- 协程挂起时，调度线程会切换到其他等待中的协程运行。
- 协程的切换是在用户态进行的，比线程间的切换更快。



协程非常适合写网络程序，可以实现**同步的编程方式**，不需要异步回调，大大减轻了程序员的思想负担。


co 协程库实现的是一种 [golang](https://github.com/golang/go/) 风格的协程，有下面几个特性：

- 内置多个调度线程，默认为系统 CPU 核数。
- 同一调度线程中的协程共用一个栈，协程挂起时，会将栈上数据 copy 出来，切换回来时再将数据 copy 到栈上。这种方式大大降低了内存占用，单机可以轻松创建数百万协程。
- 协程一旦被分配到一个调度线程，就只能在该线程中运行，而不会切换到其他调度线程运行。
- 各协程之间为平级关系，可以在任何地方(包括在协程中)创建新的协程。



co 协程库在 linux, mac, windows 等平台，分别基于 [epoll](http://man7.org/linux/man-pages/man7/epoll.7.html), [kqueue](https://man.openbsd.org/kqueue.2), [iocp](https://docs.microsoft.com/en-us/windows/win32/fileio/i-o-completion-ports) 实现。


co 协程库中 context 切换的相关代码，取自 [ruki](https://github.com/waruqi) 的 [tbox](https://github.com/tboox/tbox/)，而 tbox 则参考了 [boost](https://www.boost.org/doc/libs/1_70_0/libs/context/doc/html/index.html) 的实现，在此表示感谢！






## 协程 API




### co::go (go)
```cpp
void go(Closure* cb);
template<typename F> void go(F&& f);
template<typename F, typename P> void go(F&& f, P&& p);
template<typename F, typename T, typename P> void go(F&& f, T* t, P&& p);
```

- 此函数用于**创建协程**，与创建线程类似，需要指定一个协程函数。
- 第 1 个版本中，参数 cb 指向一个 Closure 对象，协程启动后会调用 Closure 中的 `run()` 方法。
- 第 2-4 个版本，将传入的参数打包成一个 Closure，然后调用第 1 个版本。
- 第 2 个版本中，参数 f 可以是 `void f()` 类型的函数指针，或者 `std::function<void()>` 类型的引用或指针。当 f 是 std::function<void()> 类型的指针时，用户需要保证该指针指向的函数对象在协程运行期间是存在的。
- 第 3 个版本中，有两种情况：第一种，参数 f 是 `void f(P)` 类型的函数指针，参数 p 是函数 f 的参数；第二种，参数 f 是类中不带参数的方法 `void P::f()`，参数 p 是 `P` 类型的指针。
- 第 4 个版本中，参数 f 是类中带一个参数的方法 `void T::f(P)`，参数 t 是 `T` 类型的指针，参数 p 是方法 f 的参数。
- Closure 是一个虚基类，用户可以继承 Closure 实现自己的子类，也可以用 co 提供的 `new_closure()` 方法创建 Closure 对象。new_closure() 创建的 Closure，在 run() 方法执行完时会被自动删除。
- 实际测试发现，创建 `std::function` 类型的函数对象开销较大，应尽量少用。
- 严格来说，go() 函数只是将 Closure 分配到一个调度线程中，真正创建协程是由调度线程完成的。但从用户的角度看，逻辑上可以认为 go() 创建了协程。
- 为了方便，co 使用 `using co::go;` 将此函数引入到了全局范围，在使用时无需加上前面的 namespace。



- 示例
```cpp
void f();
void g(int);
void h(int, int);
struct T {
    void f();
    void g(int);
};

T o;
std::function<void()> k(std::bind(h, 3, 7));

go(f);
go(g, 7);
go(&T::f, &o);
go(&T::g, &o, 3);
go(k);
go(&k); // The user must ensure that k is alive when the coroutine is running.
```




### co::all_schedulers
```cpp
const std::vector<xx::Scheduler*>& all_schedulers();
```

- 返回 Scheduler 列表的引用，一个 Scheduler 对应一个调度线程。
- 用户一般不需要调用此函数。





### co::coroutine_id
```cpp
int coroutine_id();
```

- 此函数返回当前协程的 id，不同协程有不同的 id。
- 此函数一般在协程中调用，在非协程中调用时，返回值是 -1。
- 协程 id 与 scheduler id 有着简单的线性对应关系。假设有 4 个 scheduler，id 分别是 0, 1, 2, 3，这些 scheduler 内的协程 id 分别是：
```cpp
4k        (0, 4, 8, ...)
4k + 1    (1, 5, 9, ...)
4k + 2    (2, 6, 10, ...)
4k + 3    (3, 7, 11, ...)
```




### co::next_scheduler
```cpp
xx::Scheduler* next_scheduler();
```

- 此函数返回下一个 Scheduler 指针。
- `go(...)` 实际上等价于 `co::next_scheduler()->add_new_task(new_closure(...))`。
- 有些应用场景中，不同的协程需要在同一个线程中运行，用户可以通过此函数返回的 scheduler，手动创建协程。



- 示例
```cpp
auto s = co::next_scheduler();
s->add_new_task(new_closure(f));     // void f();
s->add_new_task(new_closure(g, 7));  // void g(int);
```




### co::scheduler
```cpp
xx::Scheduler* scheduler();
```

- 返回当前线程对应的 scheduler 指针，如果当前线程不是调度线程，返回值是 NULL。
- 此函数一般在协程中调用，用于获取当前协程所在的 scheduler。





### co::scheduler_id
```cpp
int scheduler_id();
```

- 返回当前线程对应的 scheduler id，这个值是 0 到 `co::scheduler_num()-1` 之间的值。如果当前线程不是调度线程，返回值是 -1。
- 此函数一般在协程中调用，用于获取当前协程所在的 scheduler id。





### co::scheduler_num
```cpp
int scheduler_num();
```

- 返回 scheduler 的数量，此函数常用于实现一些协程安全的数据结构。



- 示例
```cpp
std::vector<T> v(co::scheduler_num());

void f() {
    // get object for the current scheduler
    auto& t = v[co::scheduler_id()];
}

go(f);
```




### co::sleep
```cpp
void sleep(unsigned int ms)
```

- 让当前协程睡一会儿，参数 ms 是时间，单位是毫秒。
- 此函数一般在协程中调用，在非协程中调用相当于 `sleep::ms(ms)`。





### co::stop
```cpp
void stop();
```

- 退出所有调度线程，此函数一般在程序退出时调用，用户一般不需要手动调用此函数。





### co::timeout
```cpp
bool timeout();
```

- 此函数判断之前的 `I/O` 操作是否超时。用户在调用 `co::recv()` 等带超时时间的函数后，可以调用此函数判断是否超时。
- **此函数必须在协程中调用**。





### 代码示例


```cpp
// print scheduler id and coroutine id every 3 seconds
void f() {
    while (true) {
        LOG << "s: " << co::scheduler_id() << " c: " << co::coroutine_id();
        co::sleep(3000);
    }
}

int main(int argc, char** argv) {
    flag::init(argc, argv);
    log::init();
    FLG_cout = true; // also log to terminal

    for (int i = 0; i < 32; ++i) go(f);

    while (true) sleep::sec(1024);
    return 0;
}
```






## 协程化的 socket API


co 提供了常用的协程化的 socket API，以支持基于协程的网络编程。


**大部分 API 形式上与原生的 socket API 保持一致**，这样可以减轻用户的学习负担，熟悉 socket 编程的用户可以轻松上手。


这些 API 大部分需要在协程中使用，它们在 I/O 阻塞或调用 sleep 等操作时，调度线程会挂起当前协程，切换到其他等待中的协程运行，调度线程本身并不会阻塞。借助这些 API，用户可以**轻松的实现高并发、高性能的网络程序**。

### 术语约定(必看)


**阻塞**
在描述 co 中的一些 socket API 时，会用到阻塞一词，如 accept, recv，文档中说它们会阻塞，是指当前的协程会阻塞，而当前的调度线程并不会阻塞(可以切换到其他协程运行)。用户看到的是协程，而不是调度线程，因此从用户的角度看，它们是阻塞的。实际上，这些 API 内部使用 **non-blocking socket**，并不会真的阻塞，只是在 socket 上没有数据可读或者无法立即写入数据时，调度线程会挂起当前进行 I/O 操作的协程，当 socket 变为可读或可写时，调度线程会重新唤起该协程，继续 I/O 操作。
**
**non-blocking socket**
**co 中的 socket API 必须使用 non-blocking socket**，在 windows 平台还要求 socket 支持 [overlapped I/O](https://support.microsoft.com/en-us/help/181611/socket-overlapped-i-o-versus-blocking-nonblocking-mode)，win32 API 创建的 socket 默认都支持 overlapped I/O，用户一般不需要担心这个问题。为了叙述方便，这里约定文档中说到 non-blocking socket 时，同时也表示它在 windows 上支持 overlapped I/O。

### co::socket
```cpp
sock_t socket(int domain, int type, int proto);
sock_t tcp_socket(int domain=AF_INET);
sock_t udp_socket(int domain=AF_INET);
```

- 创建 socket。
- 第 1 个函数形式上与原生 API 完全一样，在 linux 系统可以用 `man socket` 查看参数详情。
- 第 2 个函数创建一个 `TCP` socket。
- 第 3 个函数创建一个 `UDP` socket。
- 参数 domain 一般是 AF_INET 或 AF_INET6，前者表示 ipv4，后者表示 ipv6。
- **这些函数返回一个 non-blocking socket**，用户无需另外设置。发生错误时，返回值是 -1。



### co::accept
```cpp
sock_t accept(sock_t fd, void* addr, int* addrlen);
```

- 在指定 socket 上接收客户端连接，参数 fd 是之前调用 listen() 监听的 non-blocking socket，参数 addr 与 addrlen 用于接收客户端的地址信息，`*addrlen` 的初始值是 addr 所指向 buffer 的长度。如果用户不需要客户端地址信息，可以将 addr 与 addrlen 设置为 NULL。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到有新的连接进来，或者发生错误。
- 此函数成功时**返回一个 non-blocking socket**，发生错误时返回 -1。





### co::bind
```cpp
int bind(sock_t fd, const void* addr, int addrlen);
```

- 给 socket 绑定 ip 地址，参数 addr 与 addrlen 是地址信息，与原生 API 相同。
- 此函数成功时返回 0，否则返回 -1。





### co::close
```cpp
int close(sock_t fd, int ms=0);
```

- 关闭 socket，**此函数必须在 I/O 线程中**(一般是调用 send, recv 等 I/O 函数的协程中)**调用**。
- 参数 ms > 0 时，先调用 `co::sleep(ms)` 将当前协程挂起一段时间，再关闭 socket。一般只在 server 端设置 > 0 的参数，可以在一定程度上缓解非法的网络攻击。
- 此函数内部已经处理了 `EINTR` 信号，用户无需考虑。
- 此函数成功时返回 0，否则返回 -1。





### co::connect
```cpp
int connect(sock_t fd, const void* addr, int addrlen, int ms=-1);
```

- 在指定 socket 上创建到指定地址的连接，参数 fd 必须是 non-blocking 的，参数 addr 与 addrlen 是地址信息，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到连接完成，或者超时、发生错误。
- 此函数成功时返回 0，超时或发生错误返回 -1，用户可以调用 co::timeout() 查看是否超时。





### co::listen
```cpp
int listen(sock_t fd, int backlog=1024);
```

- 监听指定的 socket，参数 fd 是已经调用 bind() 绑定 ip 及端口的 socket。
- 此函数成功时返回 0，否则返回 -1。



### co::recv
```cpp
int recv(sock_t fd, void* buf, int n, int ms=-1);
```

- 在指定 socket 上接收数据，参数 fd 必须是 non-blocking 的，参数 buf 是用于接收数据的 buffer，参数 n 是 buffer 长度，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到有数据进来，或者超时、发生错误。
- 此函数成功时返回接收的数据长度(可能小于 n)，对端关闭连接时返回 0，超时或发生错误返回 -1，用户可以调用 co::timeout() 查看是否超时。



### co::recvn
```cpp
int recvn(sock_t fd, void* buf, int n, int ms=-1);
```

- 在指定 socket 上接收指定长度的数据，参数 fd 必须是 non-blocking 的，参数 buf 是用于接收数据的 buffer，参数 n 是要接收数据的长度，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到 n 字节的数据全部接收完，或者超时、发生错误。
- 此函数成功时返回 n，对端关闭连接时返回 0，超时或发生错误返回 -1，用户可以调用 co::timeout() 检查是否超时。



### co::recvfrom
```cpp
int recvfrom(sock_t fd, void* buf, int n, void* src_addr, int* addrlen, int ms=-1);
```

- 与 recv() 类似，只是可以用参数 src_addr 与 addrlen 接收源地址信息，`*addrlen` 的初始值是 src_addr 所指向 buffer 的长度，如果用户不需要源地址信息，可以将 addr 与 addrlen 设置为 NULL。
- 一般**建议只用此函数接收 UDP 数据**，对于 TCP 数据，建议用 recv() 或 recvn()。





### co::send
```cpp
int send(sock_t fd, const void* buf, int n, int ms=-1);
```

- 向指定 socket 上发送数据，参数 fd 必须是 non-blocking 的，参数 buf 与 n 是要发送的数据及长度，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此函数**必须在协程中调用**。
- 此函数会阻塞，直到 n 字节的数据全部发送完，或者超时、发生错误。
- 此函数成功时返回 n，超时或发生错误返回 -1，用户可以调用 co::timeout() 检查是否超时。



### co::sendto
```cpp
int sendto(sock_t fd, const void* buf, int n, const void* dst_addr, int addrlen, int ms=-1);
```

- 向指定的地址发送数据，当 dst_addr 为 NULL，addrlen 为 0 时，与 send() 等价。
- 一般**建议只用此函数发送 UDP 数据**，对于 TCP 数据，建议用 send()。
- fd 是 UDP socket 时，n 最大是 65507。





### co::shutdown
```cpp
int shutdown(sock_t fd, char c='b');
```

- 此函数一般用于半关闭 socket，参数 c 为 `'r'` 时表示关闭读，为 `'w'` 时表示关闭写，默认为 `'b'`，关闭读与写。
- 与 close() 一样，**此函数必须在 I/O 线程中**(一般是调用 send, recv 等 I/O 函数的协程中)**调用**。
- 此函数成功时返回 0，否则返回 -1。



### ———————————
### co::error
```cpp
int error();
```

- 返回当前的错误码。
- co 中的 socket API 返回 -1 时，可以调用此函数获取错误码。





### co::strerror
```cpp
const char* strerror(int err);
const char* strerror();
```

- 获取错误码对应的描述信息。此函数是线程安全的。
- 第 2 个版本获取当前错误的描述信息，等价于 `strerror(co::error())`。





### co::set_last_error
```cpp
void set_last_error(int err);
```

- 设置当前的错误码，用户一般不需要调用此方法。





### ———————————
### co::getsockopt
```cpp
int getsockopt(sock_t fd, int lv, int opt, void* optval, int* optlen);
```

- 获取 socket option 信息，与原生 API 完全一样，man getsockopt 看详情。





### co::setsockopt
```cpp
int setsockopt(sock_t fd, int lv, int opt, const void* optval, int optlen);
```

- 设置 socket option 信息，与原生 API 完全一样，man setsockopt 看详情。





### co::set_nonblock
```cpp
void set_nonblock(sock_t fd);
```

- 给 socket 设置 O_NONBLOCK 选项。





### co::set_reuseaddr
```cpp
void set_reuseaddr(sock_t fd);
```

- 给 socket 设置 SO_REUSEADDR 选项，一般 server 端的 listening socket 需要设置这个选项，防止 server 重启后 bind 失败。





### co::set_recv_buffer_size
```cpp
void set_recv_buffer_size(sock_t fd, int n);
```

- 设置 socket 的接收缓冲区大小，必须在 socket 连接前调用此函数。



### co::set_send_buffer_size
```cpp
void set_send_buffer_size(sock_t fd, int n);
```

- 设置 socket 的发送缓冲区大小，必须在 socket 连接前调用此函数。





### co::set_tcp_keepalive
```cpp
void set_tcp_keepalive(sock_t fd);
```

- 给 socket 设置 SO_KEEPALIVE 选项。





### co::set_tcp_nodelay
```cpp
void set_tcp_nodelay(sock_t fd);
```

- 给 socket 设置 TCP_NODELAY 选项。





### co::reset_tcp_socket
```cpp
int reset_tcp_socket(sock_t fd, int ms=0);
```

- 重置 TCP 连接，与 co::close() 类似，但主动调用方不会进入 TIME_WAIT 状态。
- 一般只有 server 端会调用此函数，用于主动关闭客户端连接，同时避免进入 TIME_WAIT 状态。





### ———————————
### co::init_ip_addr
```cpp
bool init_ip_addr(struct sockaddr_in* addr, const char* ip, int port);
bool init_ip_addr(struct sockaddr_in6* addr, const char* ip, int port);
```

- 用 ip 及 port 初始化 sockaddr 结构。
- 第 1 个版本用于 ipv4 地址，第 2 个版本用于 ipv6 地址。



- 示例
```cpp
union {
    struct sockaddr_in  v4;
    struct sockaddr_in6 v6;
} addr;

co::init_ip_addr(&addr.v4, "127.0.0.1", 7777);
co::init_ip_addr(&addr.v6, "::", 7777);
```




### co::ip_str
```cpp
fastring ip_str(const struct sockaddr_in* addr);
fastring ip_str(const struct sockaddr_in6* addr);
```

- 从 sockaddr 结构中获取 ip 字符串。
- 第 1 个版本用于 ipv4 地址，第 2 个版本用于 ipv6 地址。



- 示例
```cpp
struct sockaddr_in addr;
co::init_ip_addr(&addr, "127.0.0.1", 7777);
auto s = co::ip_str(&addr);  // s -> "127.0.0.1"
```




### co::to_string
```cpp
fastring to_string(const struct sockaddr_in* addr);
fastring to_string(const struct sockaddr_in6* addr);
fastring to_string(const void* addr, int addrlen);
```

- 将 sockaddr 地址转换成 `"ip:port"` 形式的字符串。
- 第 1 个版本用于 ipv4 地址，第 2 个版本用于 ipv6 地址。
- 第 3 个版本根据 addrlen 选择调用版本 1 或版本 2。



- 示例
```cpp
struct sockaddr_in addr;
co::init_ip_addr(&addr, "127.0.0.1", 7777);
auto s = co::to_string(&addr);  // s -> "127.0.0.1:7777"
```




### co::peer
```cpp
fastring peer(sock_t fd);
```

- 获取 peer 端的地址信息，返回值是 `"ip:port"` 形式的字符串。







## 协程同步事件




### co::Event


`co::Event` 是协程间的一种同步机制，它与线程中的 `SyncEvent` 类似。从 co 2.0.1 版本开始，co::Event 可以在线程、协程环境中混用。




#### Event::Event
```cpp
Event();
Event(Event&& e);
```

- 构造函数，第 1 个是默认构造函数，第 2 个是 move 构造函数，支持将 co::Event 放入 STL 容器中。





#### Event::signal
```cpp
void signal();
```

- 产生同步信号，co::Event 变成同步状态，所有 waiting 状态的协程会被唤醒。
- 若 co::Event 当前并没有 waiting 状态的协程，则下一个调用 wait() 方法的协程会立即返回。
- 此方法可以在任何地方调用，但一般是在协程中调用。





#### Event::wait
```cpp
void wait();
bool wait(uint32 ms);
```

- 等待同步信号，若 co::Event 当前是未同步状态，则调用的协程会进入 waiting 状态。
- 从 co 2.0.1 版本开始，此方法可以在协程或非协程中调用。
- 第 1 个版本会阻塞，直到 co::Event 变为同步状态。
- 第 2 个版本会阻塞，直到 co::Event 变为同步状态或超时，参数 ms 是超时时间，单位为毫秒。超时返回 false，正常返回 true。



### 代码示例
```cpp
co::Event ev;
int v = 0;

void f1() {
    ev.wait(100);  // wait for 100 ms
    if (v == 2) v = 1;
}

void f2() {
    v = 2;
    ev.signal();
}

go(f1);
go(f2);
ev.wait();
```






## waitgroup


**waitgroup** 是从 golang 中移植过来的一个特性，可用于等待若干协程或线程退出。




### co::WaitGroup




#### WaitGroup::WaitGroup
```cpp
WaitGroup();
WaitGroup(WaitGroup&& wg);
```

- 构造函数及移动构造函数。
- WaitGroup 对象构建完毕时，内部计数器初始值为 0。





#### WaitGroup::add
```cpp
void add(uint32 n=1);
```

- 将内部计数器加 n，参数 n 默认值是 1。





#### WaitGroup::done
```cpp
void done();
```

- 将内部计数器减 1。





#### WaitGroup::wait
```cpp
void wait();
```

- 等待内部计数器的值变为 0。





#### 代码示例
```cpp
#include "co/co.h"
#include "co/log.h"
#include "co/time.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);
    log::init();
    FLG_cout = true;

    co::WaitGroup wg;

    for (int i = 0; i < 8; ++i) {
        wg.add();
        go([&]() {
            LOG << "co: " << co::coroutine_id();
            wg.done();
        });
    }

    wg.wait();
    LOG << "wg wait end..";

    return 0;
}
```






## 协程锁




### co::Mutex


`co::Mutex` 是协程中的互斥锁，与线程中的 `Mutex` 类似，只是需要在协程环境中使用。




#### Mutex::Mutex
```cpp
Mutex();
Mutex(Mutex&& m);
```

- 构造函数，第 1 个是默认构造函数，第 2 个是 move 构造函数，可以将 co::Mutex 放入 STL 容器中。





#### Mutex::lock
```cpp
void lock();
```

- 获取锁，**必须在协程中调用**，阻塞直到获得锁为止。





#### Mutex::try_lock
```cpp
bool try_lock();
```

- 获取锁，不会阻塞，成功获取锁时返回 true，否则返回 false。
- 此方法可以在任何地方调用，但一般是在协程中调用。





#### Mutex::unlock
```cpp
void unlock();
```

- 释放锁，可以在任何地方调用，但设计良好的程序，通常是由之前获得锁的协程调用。



### co::MutexGuard


`co::MutexGuard` 与线程中的 `MutexGuard` 类似，只不过需要配合 `co::Mutex` 在协程环境中使用。




#### MutexGuard::MutexGuard
```cpp
explicit MutexGuard(co::Mutex& m);
explicit MutexGuard(co::Mutex* m);
~MutexGuard();
```

- 构造函数，调用 m.lock() 获取锁，参数 m 是 `co::Mutex` 类的引用或指针。





#### MutexGuard::~MutexGuard
```cpp
~MutexGuard();
```

- 析构函数，释放构造函数中获得的锁。





### 代码示例
```cpp
co::Mutex mtx;
int v = 0;

void f1() {
    co::MutexGuard g(mtx);
    ++v;
}

void f2() {
    co::MutexGuard g(mtx);
    --v;
}

go(f1);
go(f2);
```






## 协程池


线程支持 [TLS](https://wiki.osdev.org/Thread_Local_Storage) 机制，协程也可以支持类似的 CLS 机制，但考虑到系统中可能创建上百万协程，CLS 似乎不怎么高效，co 最终放弃了 CLS，取而代之实现了 `co::Pool` 类。




### co::Pool


co::Pool 是一种通用的协程池，它是**协程安全**的，内部存储 `void*` 类型的指针，可以用作连接池、内存池或其他用途的缓存。




#### Pool::Pool
```cpp
Pool();
Pool(Pool&& p);
Pool(std::function<void*()>&& ccb, std::function<void(void*)>&& dcb, size_t cap=(size_t)-1);
```

- 构造函数。
- 第 1 个版本是默认构造函数，与第 3 个版本相比，ccb 与 dcb 为 NULL。
- 第 2 个版本是 move 构造函数，可以将 co::Pool 放入 STL 容器中。
- 第 3 个版本中，参数 ccb 是 create callback，参数 dcb 是 destroy callback，co::Pool 在必要时用 ccb 创建元素，用 dcb 销毁元素。参数 cap 指定 co::Pool 的最大容量，默认为 -1 不限容量。
- 注意参数 cap 并不是总的容量，它是对单个线程而言。如 cap 设置为 1024，调度线程有 8 个，则总的最大容量是 8192。另外当 dcb 为 NULL 时，cap 参数会被忽略，这是因为当元素个数超过最大容量时，co::Pool 需要调用 dcb 销毁多余的元素。



- 示例
```cpp
class T;
co::Pool p(
    []() { return (void*) new T; },  // ccb, new an object of T and return the pointer
    [](void* p) { delete (T*) p; }   // dcb, delete a pointer to object of T
);
```




#### Pool::pop
```cpp
void* pop();
```

- 从 co::Pool 中弹出一个元素，**必须在协程中调用**。
- co::Pool 为空时，若 ccb 不是 NULL，则调用 ccb 创建一个元素并返回，否则直接返回 NULL。
- 此方法是协程安全的，用户不需要加锁。





#### Pool::push
```cpp
void push(void* e);
```

- 将元素 push 到 co::Pool 中，**必须在协程中调用**。
- 参数 e 为 NULL 时，直接忽略，不会放入 co::Pool。
- 由于每个线程在内部拥有自己的 pool，**用户应该在调用 pop() 的同一个线程中调用 push()**。
- 若 co::Pool 已经达到最大容量，且 dcb 不为 NULL，则直接调用 dcb 销毁元素，而不放入 pool 中。
- 此方法是协程安全的，用户不需要加锁。



- 示例
```cpp
class Redis;  // assume class Redis is a connection to the redis server
co::Pool p;

void f {
    Redis* rds = (Redis*) p.pop();     // pop a redis connection
    if (rds == NULL) rds = new Redis;
    rds->get("xx");                    // call get() method of redis
    p.push(rds);                       // push rds back to co::Pool
}

go(f);
```






### co::PoolGuard
```cpp
template<typename T> class PoolGuard;
```
`co::PoolGuard` 是一个模板类，参数 T 是 co::Pool 中指针所指向的实际类型。它构造时自动从 co::Pool 拉取元素，析构时自动将元素放回 co::Pool 中。另外，它还重载了 `operator->`，可以像智能指针一样使用它。




#### PoolGuard::PoolGuard
```cpp
explicit PoolGuard(co::Pool& pool);
explicit PoolGuard(co::Pool* pool);
```

- 构造函数，调用 pool.pop() 从 co::Pool 中拉取元素，参数 pool 是 co::Pool 类的引用或指针。





#### PoolGuard::~PoolGuard
```cpp
~PoolGuard();
```

- 析构函数，将构造函数中获取的元素，放回 co::Pool 中。





#### PoolGuard::get
```cpp
T* get() const;
```

- 获取从 co::Pool 中拉取的指针。





#### PoolGuard::operator->
```cpp
T* operator->() const;
```

- 重载 `operator->`，返回从 co::Pool 中拉取的指针。用户可以将 PoolGuard 类当指针用。





#### PoolGuard::operator bool
```cpp
explicit operator bool() const;
```

- 将 co::PoolGuard 转换为 bool 类型，若内部指针不是 NULL，返回 true，否则返回 false。





#### PoolGuard::operator!
```cpp
bool operator!() const;
```

- 判断内部指针是否为 NULL，为 NULL 时返回 true，否则返回 false。





#### PoolGuard::operator==
```cpp
bool operator==(T* p) const;
```

- 判断内部指针是否等于 p。





#### PoolGuard::operator!=
```cpp
bool operator!=(T* p) const;
```

- 判断内部指针是否不等于 p。





#### PoolGuard::operator=
```cpp
void operator=(T* p);
```

- 赋值操作，等价于 `reset(p)`。





#### PoolGuard::reset
```cpp
void reset(T* p = 0);
```

- 重置内部指针，p 必须是 `operator new` 操作创建的指针。





### 代码示例
```cpp
class Redis;  // assume class Redis is a connection to the redis server

co::Pool p(
    []() { return (void*) new Redis; },  // ccb
    [](void* p) { delete (Redis*) p; }   // dcb
);

void f() {
    co::PoolGuard<Redis> rds(p);  // now rds can be used as a Redis* pointer.
    rds->get("xx");
}

go(f);
```
上面的例子中，co::Pool 相当于 redis 连接池。如果使用 CLS 机制，一个协程一个连接，则 100 万协程需要建立 100 万连接，消耗较大。但使用 pool 机制，100 万协程可能只需要共用少量的连接。pool 机制比 CLS 更高效、更合理，这也是 co 不支持 CLS 的原因。






## I/O 事件




### co::IoEvent


`co::IoEvent` 是 co 中提供的一个基础类，前面提到的协程化的 socket API，大部分都是基于它实现的。简单来说，它可以**将非阻塞 I/O 转换为同步方式**：用户在**协程**中对一个 **non-blocking socket** 进行 I/O 操作，当 socket 不可读或不可写时，用户调用 co::IoEvent 提供的 `wait()` 方法，等待 socket 上的 I/O 事件，该协程被挂起；当 socket 变为可读或可写时，调度线程重新唤醒该协程，继续 I/O 操作。


**co/1.x** 版本并没有公开 co::IoEvent 类，只是在 co 内部使用，**co/2.0** 中将这个类公开，**方便用户将三方网络库协程化**。




#### co::io_event_t
```cpp
enum io_event_t {
    EV_read = 1,
    EV_write = 2,
};
```

- enum 类型，表示 I/O 事件类型，co::EV_read 表示读，co::EV_write 表示写。





#### IoEvent::IoEvent
```cpp
IoEvent(sock_t fd, io_event_t ev);
IoEvent(sock_t fd, int n=0);  // for windows only
```

- 构造函数，linux 与 mac 平台只提供第 1 个版本，windows 平台还提供第 2 个版本。
- 第 1 个版本中，**参数 fd 是一个 non-blocking socket**，参数 ev 是 I/O 事件类型，只能是 co::EV_read 或 co::EV_write 中的一种。调用 wait() 方法会在 socket 上等待 ev 指定的 I/O 事件，wait() 成功返回时，需要用户调用 recv, send 等函数完成 I/O 操作。**在 windows 平台，fd 必须是 TCP socket **(对于 UDP，很难用 IOCP 模拟 epoll 或 kqueue 的行为)。
- 第 2 个版本仅适用于 windows，与第 1 个版本不同，fd 可以是 UDP socket，但用户需要手动调用 WSARecvFrom, WSASendTo 等函数向 IOCP 发送 overlapped I/O 请求，然后调用 wait() 方法，当 wait() 成功返回时，表示 IOCP 已经帮用户完成了 I/O 操作。具体的用法此处不详述，代码中有详细的注释，建议直接参考 [co::IoEvent 的源码](https://github.com/idealvin/co/blob/master/include/co/co/io_event.h)，以及 windows 上 [co::accept, co::connect, co::recvfrom, co::sendto 的实现](https://github.com/idealvin/co/blob/master/src/co/sock_win.cc)。
- co::IoEvent 不支持同时等待读与写，虽然 epoll 可以同时等待读与写，但 IOCP 与 kqueue 不适合这么玩。





#### IoEvent::~IoEvent
```cpp
~IoEvent();
```

- 析构函数，从 epoll 或 kqueue 中移除之前注册的 I/O 事件。





#### IoEvent::wait
```cpp
bool wait(int ms=-1);
```

- 此方法等待 socket 上的 I/O 事件，参数 ms 是超时时间，单位为毫秒，默认为 -1，永不超时。
- 此方法阻塞，直到 I/O 事件到来，或者超时、发生错误。
- 此方法成功时返回 true，超时或发生错误时返回 false。若超时，当前错误码会被设置成 `ETIMEDOUT`，用户可以用 `co::error()` 获取错误码，也可以直接用 `co::timeout()` 判断是否超时。





#### 代码示例
```cpp
int recv(sock_t fd, void* buf, int n, int ms) {
    CHECK(gSched) << "must be called in coroutine..";
    IoEvent ev(fd, EV_read);

    do {
        int r = (int) raw_recv(fd, buf, n, 0);
        if (r != -1) return r;

        if (errno == EWOULDBLOCK || errno == EAGAIN) {
            if (!ev.wait(ms)) return -1;
        } else if (errno != EINTR) {
            return -1;
        }
    } while (true);
}
```
上面的例子是 co::recv 的实现，调用原生 recv 方法产生 EWOULDBLOCK 或 EAGAIN 错误时，用 co::IoEvent 等待读事件，wait() 正常返回时表示 socket 可读，继续调用原生 recv 方法完成读操作。






## 协程中使用三方网络库


在协程中直接使用三方网络库时，有可能阻塞调度线程，导致调度线程无法正常工作。解决这个问题有两种方法，第一种是将三方库协程化，第二种是 hook 系统中的 socket API，下面分别介绍。

### 协程化


协程化需要三方库提供非阻塞 API，利用 co::IoEvent 可以轻松将这些 API 转换为协程同步方式。
```cpp
int recv(SSL* s, void* buf, int n, int ms) {
    CHECK(co::scheduler()) << "must be called in coroutine..";
    int r, e;
    int fd = SSL_get_fd(s);
    if (fd < 0) return -1;

    do {
        ERR_clear_error();
        r = SSL_read(s, buf, n);
        if (r > 0) return r; // success
        if (r == 0) {
            DLOG << "SSL_read return 0, error: " << SSL_get_error(s, 0);
            return 0;
        }
 
        e = SSL_get_error(s, r);
        if (e == SSL_ERROR_WANT_READ) {
            co::IoEvent ev(fd, co::EV_read);
            if (!ev.wait(ms)) return -1;
        } else if (e == SSL_ERROR_WANT_WRITE) {
            co::IoEvent ev(fd, co::EV_write);
            if (!ev.wait(ms)) return -1;
        } else {
            DLOG << "SSL_read return " << r << ", error: " << e;
            return r;
        }
    } while (true);
}
```
上面是将 openssl 中的 SSL_read 协程化的例子，整个过程比较简单，底层使用 non-blocking socket，在 SSL_read 产生 `SSL_ERROR_WANT_READ` 错误时，用 co::IoEvent 等待读事件，产生 `SSL_ERROR_WANT_WRITE` 错误时，用 co::IoEvent 等待写事件，wait() 正常返回时，表示 socket 可读或可写，继续调用 SSL_read 完成 I/O 操作。


目前，co 已经成功将 openssl, libcurl 协程化。理论上，支持非阻塞 I/O 操作的三方网络库，都可以用与上面类似的方法协程化。希望有更多的朋友加入到这个行列(我个人实在精力有限...)，将常用的网络库协程化，打造良好的协程编程生态。



### 系统 API hook


API hook 简单来说就是拦截系统 API 请求，如果发现该请求是在协程中，且使用 blocking socket，就将 socket 修改成 non-blocking 模式，然后利用 co::IoEvent 或 co 中更底层的接口，等待 socket 上的 I/O 事件，I/O 事件到来时，再唤醒协程，调用原生的 socket api 完成 I/O 操作。


从 co 2.0.1 开始，在 linux/mac/windows 平台均已支持 hook。


**API hook 与协程化的区别**在于：前者是将阻塞 API 转换成协程同步方式，后者是将非阻塞 API 转换成协程同步方式。协程同步方式是指协程可能会阻塞，从协程的角度看是同步的，但协程调度线程不会阻塞，它可以切换到其他协程运行。另外，它们的使用方式也是不同的，前者需要在协程中用阻塞的方式调用原生 API，后者则直接在协程中调用协程化的 API。

API hook 的好处在于，只需要 hook 系统中的少量 socket API，就可以在协程中使用所有提供阻塞 API 的三方库。协程化则需要为每个三方库各提供一套协程化的 API，但它比 API hook 性能更好，且更安全，可以避免由三方库的复杂性引起的一些问题。






## 基于协程的网络编程模式


协程可以用同步的方式，实现高并发、高性能的网络程序。协程虽然会阻塞，但调度线程可以在大量的协程间快速切换，因此要实现高并发，只需要创建更多的协程即可。


以 TCP 程序为例，服务端一般采用一个连接一个协程的模式，为每个客户端连接创建新的协程，在协程中处理连接上的数据。客户端没必要一个连接一个协程，一般使用连接池，多个协程共用连接池中的连接。




### 服务端网络模型
```cpp
// recv or send data on the connection
void on_connection(int fd) {
    while (true) {
        co::recv(fd, ...);  // recv request from client
        process(...);       // process the request
        co::send(fd, ...);  // send response to client
    }
}

void server_fun() {
    while (true) {
        int fd = co::accept(...);
        if (fd != -1) go(on_connection, fd);
    }
}

go(server_fun);
```

- 在一个协程中，调用 `co::accept()` 接受客户端连接。
- 有连接到来时，创建一个新的协程，在协程中处理连接上的数据。
- `on_connection()` 是处理连接数据的协程函数，接收请求、处理请求、发送结果，在该协程中以完全同步的方式进行，不需要任何异步回调。
- 完整的实现可以参考 [co 中的测试代码](https://github.com/idealvin/co/blob/master/test/so/tcp2.cc)。





### 客户端网络模型
```cpp
void client_fun() {
    while true {
        if (!connected) co::connect(...);  // connect to the server
        co::send(...);                     // send request to the server
        co::recv(...);                     // recv response from the server
        process(...);                      // process the response
        if (over) co::close(...);          // close the connection
    }
}

go(client_fun);
```

- 建立连接、发送请求、接收响应、处理响应，在协程中以完全同步的方式进行，**关闭连接也必须在同一个协程中进行**。
- 完整的实现可以参考 [co 中的测试代码](https://github.com/idealvin/co/blob/master/test/so/tcp2.cc)。



实际应用中，一般使用 **co::Pool** 作为连接池，以避免创建过多的连接：
```cpp
co::Pool pool;

void client_fun() {
    while true {
        co::PoolGuard<Connection> conn(pool);  // get a idle connection from the pool
        conn->send(...);                       // send request to the server
        conn->recv(...);                       // recv response from the server
        process(...);                          // process the response
        if (over) conn->close(...);            // close the connection
    }
}

go(client_fun);
```

- co::PoolGuard 构造时自动从 co::Pool 中获取一个空闲连接，析构时自动将该连接放加 co::Pool 中。







## 配置




### co_sched_num
```cpp
DEF_uint32(co_sched_num, os::cpunum(), "#1 number of coroutine schedulers, default: os::cpunum()");
```

- 协程调度线程的数量，默认为系统 CPU 核数。目前的实现中，这个值最大也是系统 CPU 核数。





### co_stack_size
```cpp
DEF_uint32(co_stack_size, 1024 * 1024, "#1 size of the stack shared by coroutines, default: 1M");
```

- 协程栈大小，默认为 1M，同一调度线程中的协程共用一个栈。


