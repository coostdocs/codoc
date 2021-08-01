---
weight: 11
title: "Coroutine"
---

include: [co/co.h](https://github.com/idealvin/co/blob/master/include/co/co.h).


## Basic concepts


- Coroutines are lightweight scheduling units that run in threads.
- Coroutines are to threads, similar to threads to processes.
- There can be multiple threads in a process and multiple coroutines in a thread.
- The thread where the coroutine runs in is generally called the **scheduling thread**.
- The scheduling thread will suspend a coroutine, if it was blocked on an I/O operation or sleep was called in the coroutine.
- When a coroutine is suspended, the scheduling thread will switch to other coroutines waiting to be executed.
- Switching of coroutines is done in user mode, which is faster than switching between threads.



Coroutines are very suitable for network programming, and can achieve synchronous programming without asynchronous callbacks, which greatly reduces the programmer's mental burden. 


co implements a [golang](https://github.com/golang/go/) style coroutine with the following features:


- Built-in multiple scheduling threads, the default is number of the system CPU cores.
- The coroutines in the same scheduling thread share a stack. When a coroutine is suspended, the data on the stack will be copied out, and the data will be copied onto the stack again when it is switched back. This method greatly reduces the memory footprint, and a single machine can easily create millions of coroutines.
- There is a flat relationship between coroutines, and new coroutines can be created from anywhere (including in coroutines).



The coroutine library is based on [epoll](http://man7.org/linux/man-pages/man7/epoll.7.html), [kqueue](https://man.openbsd.org/kqueue.2), [iocp](https://docs.microsoft.com/en-us/windows/win32/fileio/io-completion-ports). The relevant code for context switching is taken from [tbox](https://github.com/tboox/tbox/) by [ruki](https://github.com/waruqi), and tbox refers to the implementation of [boost](https://www.boost.org/doc/libs/1_70_0/libs/context/doc/html/index.html), thanks here!




## Coroutine API


### co::go (go)


```cpp
void go(Closure* cb);
template<typename F> void go(F&& f);
template<typename F, typename P> void go(F&& f, P&& p);
template<typename F, typename T, typename P> void go(F&& f, T* t, P&& p);
```


- This function is used to **create a coroutine**, similar to creating a thread, a coroutine function must be specified.
- In the first version, the parameter cb points to a Closure object. When the coroutine is started, the `run()` method in the Closure will be called.
- The 2-4th version, pack the incoming parameters into a Closure, and then call the first version.
- In the second version, the parameter f can be a function pointer of type `void f()`, or a reference or pointer of type `std::function<void()>`. When f is a pointer of type std::function<void()>, the user must ensure that the function object pointed to by the pointer exists during the running of the coroutine.
- In the third version, there are two cases: the first, the parameter f is a function pointer of type `void f(P)`, and p is the parameter of f; the second, the parameter f is a method in class P, and p is a pointer to class `P`.
- In the fourth version, the parameter f is a method with one parameter in the class T, the parameter t is a pointer to class `T`, and p is the parameter of f.
- Closure is a virtual base class. Users can inherit Closure to implement their own subclasses, or use `new_closure()` provided by co to create Closure objects. The Closure created by new_closure() will be automatically deleted when the run() method is done.
- Creating a object of type `std::function` is expensive and should be used as little as possible.
- Strictly speaking, the go() function just assigns a Closure to a scheduling thread, and the actual creation of the coroutine is done by the scheduling thread. But from the user's point of view, logically it can be considered that go() creates a coroutine.
- For convenience, co uses `using co::go;` to introduce this function to the global scope, and there is no need to add the preceding namespace when using it.



- Example



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


- Return a reference to the Scheduler list, one Scheduler corresponds to one scheduling thread.
- Generally, users do not need to call this function.





### co::coroutine_id


```cpp
int coroutine_id();
```


- This function returns the id of the current coroutine. Different coroutines have different ids.
- This function is generally called in a coroutine. When called in a non-coroutine, the return value is -1.
- There is a simple linear correspondence between the coroutine id and the scheduler id. Assuming there are 4 schedulers, the ids are 0, 1, 2, 3, and the coroutine ids in these schedulers are:



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

- This function returns the next Scheduler pointer.
- `go(...)` is actually equivalent to `co::next_scheduler()->add_new_task(new_closure(...))`.
- In some cases, different coroutines need to run in the same thread. Users can manually create coroutines through the scheduler returned by this function.



- Example
```cpp
auto s = co::next_scheduler();
s->add_new_task(new_closure(f));     // void f();
s->add_new_task(new_closure(g, 7));  // void g(int);
```




### co::scheduler


```cpp
xx::Scheduler* scheduler();
```


- Return the scheduler pointer of the current thread. If the current thread is not a scheduling thread, the return value is NULL.
- This function is generally called in a coroutine to obtain the scheduler where it runs in.





### co::scheduler_id


```cpp
int scheduler_id();
```


- Returns the scheduler id of the current thread. This value is between 0 and `co::scheduler_num()-1`. If the current thread is not a scheduling thread, the return value is -1.
- This function is generally called in the coroutine to obtain the id of the scheduler where it runs in.





### co::scheduler_num


```cpp
int scheduler_num();
```


- Returns the number of schedulers. This function is usually used to implement some coroutine-safe data structures.



- Example



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


- Let the current coroutine sleep for a while, the parameter ms is time in milliseconds.
- This function is generally called in a coroutine, calling in a non-coroutine is equivalent to `sleep::ms(ms)`.





### co::stop


```cpp
void stop();
```


- Stop all scheduling threads. This function is generally called when the program exits, and users generally do not need to manually call this function.





### co::timeout


```cpp
bool timeout();
```


- This function checks whether the previous `I/O` operation has timed out. After users call an I/O function like `co::recv()` with a timeout, they can call this function to determine whether it has timed out.
- **This function must be called in the coroutine**.





### Code example


```cpp
// print scheduler id and coroutine id every 3 seconds
void f() {
    while (true) {
        LOG << "s: "<< co::scheduler_id() <<" c: "<< co::coroutine_id();
        co::sleep(3000);
    }
}

int main(int argc, char** argv) {
    flag::init(argc, argv);
    log::init();
    FLG_cout = true; // also log to terminal

    for (int i = 0; i <32; ++i) go(f);

    while (true) sleep::sec(1024);
    return 0;
}
```






## Coroutineized socket API


co provides commonly used coroutineized socket APIs to support coroutine-based network programming. 


**Most of the APIs are consistent in form with the native socket APIs**, which can reduce the user's learning burden, and users who are familiar with socket programming can easily get started. 


Most of these APIs need to be used in coroutines. When they are blocked on I/O or sleep, the scheduling thread will suspend the current coroutine and switch to other waiting coroutines, and the scheduling thread itself will not block. With these APIs, users can easily implement **high-concurrency and high-performance network programs**.




### Terminology convention (must see)




**Blocking**
**
When describing some socket APIs in co, the term blocking is used, such as accept, recv. The document says that they will block, which means that the current coroutine will block , but the current scheduling thread will not block (it can switch to other coroutines to run). The user sees the coroutine, not the scheduling thread, so from the user's point of view, they are blocking. In fact, these APIs use **non-blocking socket** internally, which does not really block, but when there is no data to read on the socket or data cannot be written immediately, the scheduling thread will suspend the current coroutine. When the socket becomes readable or writable, the scheduling thread will awaken the coroutine again and continue the I/O operation. 
**
**
**non-blocking socket**
**
The socket API in co must use **non-blocking socket**. On windows, socket must also support [overlapped I/O](https://support.microsoft.com/en-us/help/181611/socket-overlapped-io-versus-blocking-nonblocking-mode), which is supported by default for sockets created with win32 API, users generally no need to worry about it. For narrative convenience, when non-blocking socket is mentioned in the document, it also means that it supports overlapped I/O on windows.




### co::socket


```cpp
sock_t socket(int domain, int type, int proto);
sock_t tcp_socket(int domain=AF_INET);
sock_t udp_socket(int domain=AF_INET);
```


- Create a socket.
- The form of the first function is exactly the same as the native API. You can use `man socket` to see the parameter details on linux.
- The second function creates a TCP socket.
- The third function creates a UDP socket.
- The parameter domain is usually AF_INET or AF_INET6, the former means ipv4 and the latter means ipv6.
- **These functions return a non-blocking socket**. When an error occurs, the return value is -1.





### co::accept


```cpp
sock_t accept(sock_t fd, void* addr, int* addrlen);
```


- Receive the client connection on the specified socket, the parameter fd is a non-blocking socket, and the parameters addr and addrlen are used to receive the client's address information. The initial value of `*addrlen` is the length of the buffer pointed to by addr. If the user does not need the client address information, addr and addrlen should be set to NULL.
- This function **must be called in the coroutine**.
- This function will block until a new connection comes in or an error occurs.
- This function **returns a non-blocking socket** on success, and returns -1 when an error occurs.





### co::bind


```cpp
int bind(sock_t fd, const void* addr, int addrlen);
```


- Bind the ip address to the socket, the parameters addr and addrlen are the address information, which is the same as the native API.
- This function returns 0 on success, otherwise returns -1.





### co::close


```cpp
int close(sock_t fd, int ms=0);
```


- Close the socket, **this function must be called in the I/O thread** (usually in the coroutine that calls send, recv and other I/O functions).
- When the parameter ms > 0, first call `co::sleep(ms)` to suspend the current coroutine for a period of time, and then close the socket. 
- The** EINTR** signal has been processed internally in this function, and the user does not need to consider it.
- This function returns 0 on success, otherwise it returns -1.





### co::connect


```cpp
int connect(sock_t fd, const void* addr, int addrlen, int ms=-1);
```


- Create a connection to the specified address on the specified socket, the parameter fd must be non-blocking,  the parameter ms is the timeout period in milliseconds, the default is -1, which will never time out.
- This function **must be called in the coroutine**.
- This function will block until the connection is completed, or timeout or an error occurs.
- This function returns 0 on success, and returns -1 on timeout or an error occurs. The user can call co::timeout() to check whether it has timed out.





### co::listen


```cpp
int listen(sock_t fd, int backlog=1024);
```


- Listenning on the specified socket.
- This function returns 0 on success, otherwise it returns -1.





### co::recv


```cpp
int recv(sock_t fd, void* buf, int n, int ms=-1);
```


- Receive data on the specified socket, the parameter fd must be non-blocking, the parameter buf is the buffer used to receive the data, the parameter n is the buffer length, and the parameter ms is the timeout period in milliseconds, the default is -1, never time out.
- This function **must be called in the coroutine**.
- This function will block until any data comes in, or timeout or any error occurs.
- This function returns length of the data received (may be less than n) on success, returns 0 when the peer closes the connection, returns -1 when timeout or an error occurs, and the user can call co::timeout() to check whether it has timed out.





### co::recvn


```cpp
int recvn(sock_t fd, void* buf, int n, int ms=-1);
```


- Receive data of the specified length on the specified socket, the parameter fd must be non-blocking, the parameter ms is the timeout period in milliseconds, the default is -1, never timeout.
- This function **must be called in the coroutine**.
- This function will block until all n bytes of data are received, or timeout or an error occurs.
- This function returns n on success, returns 0 when the peer closes the connection, and returns -1 when timeout or an error occurs. The user can call co::timeout() to check whether it has timed out.





### co::recvfrom


```cpp
int recvfrom(sock_t fd, void* buf, int n, void* src_addr, int* addrlen, int ms=-1);
```


- Similar to recv(), except that the parameters src_addr and addrlen can be used to receive the source address information. The initial value of `*addrlen` is the length of the buffer pointed to by src_addr. If the user does not need the source address information, addr and addrlen should be set to NULL .
- Generally** it is recommended to only use this function to receive UDP data**, for TCP data, it is recommended to use recv() or recvn().





### co::send


```cpp
int send(sock_t fd, const void* buf, int n, int ms=-1);
```


- Send data to the specified socket, the parameter fd must be non-blocking, the parameter ms is the timeout period in milliseconds, the default is -1, which will never time out.
- This function **must be called in the coroutine**.
- This function will block until all n bytes of data are sent, or timeout or an error occurs.
- This function returns n on success, and returns -1 on timeout or an error occurs. The user can call co::timeout() to check whether it has timed out.





### co::sendto


```cpp
int sendto(sock_t fd, const void* buf, int n, const void* dst_addr, int addrlen, int ms=-1);
```


- Send data to the specified address. When dst_addr is NULL and addrlen is 0, it is equivalent to send().
- Generally **it is recommended to only use this function to send UDP data**, for TCP data, it is recommended to use send().
- When fd is a UDP socket, the maximum n is 65507.





### co::shutdown


```cpp
int shutdown(sock_t fd, char c='b');
```


- This function is generally used to half-close the socket. The parameter c is a hint,  **'r'** for read, **'w' **for write,  the default is **'b'**, which means both reading and writing are closed.
- Like close(), **this function must be called in the I/O thread** (usually in the coroutine that calls send, recv or other I/O functions).
- This function returns 0 on success, otherwise it returns -1.





### co::error


```cpp
int error();
```


- Return the current error code.
- When the socket API in co returns -1, the user can call this function to get the error code.





### co::strerror


```cpp
const char* strerror(int err);
const char* strerror();
```


- Get the error string corresponding to the error code. It is thread safe.
- The second version gets the description information of the current error, which is equivalent to `strerror(co::error())`.





### co::set_last_error


```cpp
void set_last_error(int err);
```


- Set the current error code, users generally do not need to call this function.





### ———————————




### co::getsockopt


```cpp
int getsockopt(sock_t fd, int lv, int opt, void* optval, int* optlen);
```


- Get socket option information, which is exactly the same as native API, **man getsockopt** for details.





### co::setsockopt


```cpp
int setsockopt(sock_t fd, int lv, int opt, const void* optval, int optlen);
```


- Set the socket option information, which is exactly the same as the native API, **man setsockopt** for details.





### co::set_nonblock


```cpp
void set_nonblock(sock_t fd);
```


- Set the **O_NONBLOCK **option to the socket.





### co::set_reuseaddr


```cpp
void set_reuseaddr(sock_t fd);
```


- Set the **SO_REUSEADDR **option for the socket. 
- Generally, the listening socket of a server needs to set this option, or the bind() may fail when the server restarts.





### co::set_recv_buffer_size


```cpp
void set_recv_buffer_size(sock_t fd, int n);
```


- Set the receiving buffer size of the socket. This function must be called before the socket is connected.





### co::set_send_buffer_size


```cpp
void set_send_buffer_size(sock_t fd, int n);
```


- Set the size of the sending buffer of the socket. This function must be called before the socket is connected.





### co::set_tcp_keepalive


```cpp
void set_tcp_keepalive(sock_t fd);
```


- Set the **SO_KEEPALIVE **option to the socket.





### co::set_tcp_nodelay


```cpp
void set_tcp_nodelay(sock_t fd);
```


- Set the **TCP_NODELAY **option to the socket.





### co::reset_tcp_socket


```cpp
int reset_tcp_socket(sock_t fd, int ms=0);
```


- Reset a TCP connection, similar to co::close(), but the caller will not enter the TIME_WAIT state.
- Generally, only the server side will call this function to close a client connection without entering the TIME_WAIT state.





### ———————————




### co::init_ip_addr


```cpp
bool init_ip_addr(struct sockaddr_in* addr, const char* ip, int port);
bool init_ip_addr(struct sockaddr_in6* addr, const char* ip, int port);
```


- Initialize the sockaddr structure with ip and port.
- The first version is used for ipv4 addresses, and the second version is used for ipv6 addresses.



- Example



```cpp
union {
    struct sockaddr_in v4;
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


- Get the ip string from the sockaddr structure.
- The first version is used for ipv4 addresses, and the second version is used for ipv6 addresses.



- Example



```cpp
struct sockaddr_in addr;
co::init_ip_addr(&addr, "127.0.0.1", 7777);
auto s = co::ip_str(&addr); // s -> "127.0.0.1"
```




### co::to_string


```cpp
fastring to_string(const struct sockaddr_in* addr);
fastring to_string(const struct sockaddr_in6* addr);
fastring to_string(const void* addr, int addrlen);
```


- Convert the sockaddr address into a string in the form of `"ip:port"`.
- The first version is used for ipv4 addresses, and the second version is used for ipv6 addresses.
- The third version calls version 1 or version 2 according to addrlen.



- Example



```cpp
struct sockaddr_in addr;
co::init_ip_addr(&addr, "127.0.0.1", 7777);
auto s = co::to_string(&addr); // s -> "127.0.0.1:7777"
```




### co::peer


```cpp
fastring peer(sock_t fd);
```


- Get the address information of the peer. The return value is a string in the form of `"ip:port"`.







## Coroutine synchronization event




### co::Event


`co::Event` is a synchronization mechanism between coroutines. It is similar to `SyncEvent` in threads. Since co 2.0.1, co::Event can be used in both coroutines and non-coroutines. 




#### Event::Event


```cpp
Event();
Event(Event&& e);
```


- Constructor, the first is the default constructor, and the second is the move constructor, which supports putting co::Event into the STL container.





#### Event::signal


```cpp
void signal();
```


- Generate a signal, and co::Event turns to synchronized state, all waiting coroutines will be awaken.
- If co::Event currently has no waiting coroutine, the next coroutine that calls the wait() method will return immediately.
- This method can be called anywhere, but it is usually called in a coroutine.





#### Event::wait


```cpp
void wait();
bool wait(unsigned int ms);
```


- Wait for the synchronization signal. If co::Event is currently unsynchronized, the calling coroutine will enter a waiting state.
- Since co 2.0.1, it can be called in both coroutines and non-coroutines.
- The first version will block until co::Event becomes synchronized.
- The second version will block until co::Event becomes synchronized or timed out. The parameter ms is the timeout period in milliseconds. It returns false when timeout, otherwise returns true.



### Code example


```cpp
int v;

void f1() {
    ev.wait(100); // wait for 100 ms
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


**waitgroup** is a feature from golang. It can be used to wait for several coroutines or threads to exit.




### co::WaitGroup


#### WaitGroup::WaitGroup
```cpp
WaitGroup();
WaitGroup(WaitGroup&& wg);
```

- Constructor and move constructor.
- When the WaitGroup object is constructed, the initial value of the internal counter is 0.





#### WaitGroup::add
```cpp
void add(uint32 n=1);
```

- Increase the internal counter by n, the default value of parameter n is 1.





#### WaitGroup::done
```cpp
void done();
```

- Decrement the internal counter by 1.





#### WaitGroup::wait
```cpp
void wait();
```

- Wait until the value of the internal counter becomes 0.





#### Code example
```cpp
#include "co/co.h"
#include "co/log.h"
#include "co/time.h"
int main(int argc, char** argv) {
     flag::init(argc, argv);
     log::init();
     FLG_cout = true;
    
     co::WaitGroup wg;
     for (int i = 0; i <8; ++i) {
         wg.add();
         go([&]() {
             LOG << "co: "<< co::coroutine_id();
             wg.done();
         });
     }
    
     wg.wait();
     LOG << "wg wait end..";
     return 0;
}
```




## Coroutine lock


### co::Mutex


`co::Mutex` is the mutex lock for coroutines, similar to `Mutex` for threads, but needs to be used in coroutine environments. 




#### Mutex::Mutex


```cpp
Mutex();
Mutex(Mutex&& m);
```


- Constructor, the first is the default constructor, and the second is the move constructor. You can put co::Mutex into the STL container.





#### Mutex::lock


```cpp
void lock();
```


- Acquire the lock, **must be called** in the coroutine.
- It blocks until the lock is acquired.





#### Mutex::try_lock


```cpp
bool try_lock();
```


- Acquire the lock, will not block. It returns true when the lock is successfully acquired, otherwise it returns false.
- This method can be called anywhere, but it is usually called in a coroutine.





#### Mutex::unlock


```cpp
void unlock();
```


- Release the lock, which can be called from anywhere, but a well-designed program is usually called by the coroutine that held the lock.





### co::MutexGuard


`co::MutexGuard` is similar to `MutexGuard` in threads, except that it needs to be used in coroutine environments with `co::Mutex`. 




#### MutexGuard::MutexGuard


```cpp
explicit MutexGuard(co::Mutex& m);
explicit MutexGuard(co::Mutex* m);
~MutexGuard();
```


- Constructor, call m.lock() to acquire the lock, the parameter m is a reference or pointer of the `co::Mutex`.





#### MutexGuard::~MutexGuard


```cpp
~MutexGuard();
```


- Destructor, release the lock acquired in the constructor.





### Code example


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






## Coroutine Pool


Thread support [TLS](https://wiki.osdev.org/Thread_Local_Storage) mechanism, coroutines can also support similar CLS mechanism, but considering that millions of coroutines may be created in the system, CLS seems to be not so efficient, co eventually abandoned CLS and implemented the `co::Pool` instead. 




### co::Pool


co::Pool is a general coroutine pool, which is **coroutine safe**. It stores pointers of type `void*` internally, which can be used as connection pool, memory pool or cache for other purposes. 




#### Pool::Pool


```cpp
Pool();
Pool(Pool&& p);
Pool(std::function<void*()>&& ccb, std::function<void(void*)>&& dcb, size_t cap=(size_t)-1);
```


- Constructor.
- The first version is the default constructor. Compared with the third version, ccb and dcb are NULL.
- The second version is the move constructor.
- In the third version, the parameter ccb is a create callback, the parameter dcb is a destroy callback, co::Pool uses ccb to create an element when necessary, and dcb to destroy an element. The parameter cap specifies the maximum capacity of co::Pool, and the default is -1 for unlimited.
- Note that the parameter cap is not the total capacity, but for a single thread. If cap is set to 1024 and there are 8 scheduling threads, the total maximum capacity is 8192. In addition, when dcb is NULL, the cap parameter will be ignored. This is because when the number of elements exceeds the maximum capacity, co::Pool needs to call dcb to destroy the extra elements.



- Example



```cpp
class T;
co::Pool p(
    []() {return (void*) new T; },  // ccb, new an object of T and return the pointer
    [](void* p) {delete (T*) p;},   // dcb, delete a pointer to object of T
    1024                            // cap
);
```




#### Pool::pop


```cpp
void* pop();
```


- Pop an element from co::Pool, **must be called in the coroutine**.
- When co::Pool is empty, if ccb is not NULL, call ccb() to create an element, otherwise it will return NULL.
- This method is coroutine safe, users do not need a lock.





#### Pool::push


```cpp
void push(void* e);
```


- Push an element to co::Pool, **must be called in the coroutine**.
- If e is NULL, it will be ignored.
- As each thread has its own pool, push() **SHOULD** be called in the same thread that calls pop().
- If co::Pool has reached the maximum capacity, and dcb is not NULL, call dcb(e) to destroy the element.
- This method is coroutine safe, users do not need a lock.



- Example



```cpp
class Redis; // assume class Redis is a connection to the redis server
co::Pool p;

void f {
    Redis* rds = (Redis*) p.pop(); // pop a redis connection
    if (rds == NULL) rds = new Redis;
    rds->get("xx");  // call get() method of redis
    p.push(rds);     // push rds back to co::Pool
}

go(f);
```




#### Pool::size
```cpp
size_t size() const;
```

- Returns size of the pool for the current thread.





### co::PoolGuard


```cpp
template<typename T> class PoolGuard;
```


`co::PoolGuard` is a template class, and the parameter T is the actual type pointed to by the pointer in co::Pool. It automatically pops an element from co::Pool during construction, and automatically pushs it back into co::Pool during destruction. In addition, it also overloads `operator->`, so it can be used like a smart pointer. 




#### PoolGuard::PoolGuard


```cpp
explicit PoolGuard(co::Pool& pool);
explicit PoolGuard(co::Pool* pool);
```


- Constructor, call pool.pop() to pop an element from co::Pool.





#### PoolGuard::~PoolGuard


```cpp
~PoolGuard();
```


- Destructor, push the element back into co::Pool.





#### PoolGuard::get


```cpp
T* get() const;
```


- Get the pointer obtained in the constructor.





#### PoolGuard::operator->


```cpp
T* operator->() const;
```


- Overload `operator->`, returns the pointer obtained in the constructor.





#### PoolGuard::operator bool


```cpp
explicit operator bool() const;
```


- Convert co::PoolGuard to bool type, if the internal pointer is not NULL, returns true, otherwise returns false.





#### PoolGuard::operator!


```cpp
bool operator!() const;
```


- Check whether the internal pointer is NULL, return true if it is NULL, otherwise returns false.





#### PoolGuard::operator==


```cpp
bool operator==(T* p) const;
```


- Determine whether the internal pointer is equal to p.





#### PoolGuard::operator!=


```cpp
bool operator!=(T* p) const;
```


- Determine whether the internal pointer is not equal to p.





#### PoolGuard::operator=


```cpp
void operator=(T* p);
```


- Assignment, equivalent to `reset(p)`.





#### PoolGuard::reset


```cpp
void reset(T* p = 0);
```


- Reset the internal pointer, p must be a pointer created by the `operator new`.





### Code example


```cpp
class Redis; // assume class Redis is a connection to the redis server

co::Pool p(
    []() {return (void*) new Redis; }, // ccb
    [](void* p) {delete (Redis*) p;}   // dcb
);

void f() {
    co::PoolGuard<Redis> rds(p); // now rds can be used like a Redis* pointer.
    rds->get("xx");
}

go(f);
```






## I/O events


### co::IoEvent


`co::IoEvent` is a basic class provided in co. Most of the aforementioned coroutineized socket APIs are implemented based on it. In simple terms, it can **convert non-blocking I/O to synchronous mode**: the user performs I/O operations on a **non-blocking socket** in the **coroutine**, when the socket is unreadable or unwritable, the user calls the `wait()` method of co::IoEvent and waits for I/O events on the socket, and the coroutine is suspended. When the socket becomes readable or writable, the scheduling thread resume the coroutine and continue the I/O operation. 


**co/1.x** does not expose the co::IoEvent, but it is only used internally. This class is public in **co/2.0**, which is convenient for users to coroutineize third-party network libraries. 




#### co::io_event_t


```cpp
enum io_event_t {
    EV_read = 1,
    EV_write = 2,
};
```


- enum type, which means I/O event type, co::EV_read for read, co::EV_write for write.





#### IoEvent::IoEvent


```cpp
IoEvent(sock_t fd, io_event_t ev);
IoEvent(sock_t fd, int n=0); // for windows only
```


- Constructor, linux and mac platforms only provide the first version, windows platform also provides the second version.
- In the first version, the **parameter fd is a non-blocking socket**, and the parameter ev is an I/O event, which is one of co::EV_read or co::EV_write. Calling the wait() method will wait for the I/O event specified by ev on the socket. When wait() returns successfully, the user needs to call recv, send or other I/O functions to complete the I/O operation. **On windows, fd must be a TCP socket **(For UDP, it is difficult to simulate the behavior of epoll or kqueue with IOCP).
- The second version is only applicable to windows. Unlike the first version, fd can be a UDP socket, but the user must manually call WSARecvFrom, WSASendTo or other functions to post an overlapped I/O operation to IOCP, and then call the wait() method. When wait() returns successfully, it means that IOCP has completed the I/O operation for the user. The specific usage is not detailed here. There are detailed comments in the code. It is recommended to directly refer to the source codes of [co::IoEvent](https://github.com/idealvin/co/blob/master/include/co/co/io_event.h), and implementation of [co::accept, co::connect, co::recvfrom, co::sendto](https://github.com/idealvin/co/blob/master/src/co/sock_win.cc) on windows.
- co::IoEvent does not support waiting for both reading and writing at the same time. Although epoll can do so, IOCP and kqueue are not suitable for playing like this.





#### IoEvent::~IoEvent


```cpp
~IoEvent();
```


- Destructor, remove previously registered I/O events from epoll or kqueue.





#### IoEvent::wait


```cpp
bool wait(int ms=-1);
```


- This method waits for I/O events on the socket. The parameter ms is the timeout period in milliseconds, the default is -1, which will never time out.
- This method blocks until the I/O event arrives, or timeout or an error occurs.
- This method returns true on success, and returns false when timeout or an error occurs. If timeout, the current error code will be set to `ETIMEDOUT`, and the user can use `co::error()` to get the error code, or directly use `co::timeout()` to check whether it has timed out.





#### Code example


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


The above example is the implementation of co::recv. When the native recv() generates an EWOULDBLOCK or EAGAIN error, use co::IoEvent to wait for the read event. When wait() returns normally, the socket is readable, continue to call the native recv() to complete the read operation . 




## Use third-party network libraries in coroutine


When a third-party network library is used directly in coroutine, it may block the scheduling thread and the scheduling thread will not work normally. There are two ways to solve this problem. The first is to coroutineize third-party libraries, and the second is to hook the system socket API.




### Coroutineization


Coroutineization requires third-party libraries to provide non-blocking APIs, and co::IoEvent can be used to easily coroutineize these APIs.


```cpp
int recv(SSL* s, void* buf, int n, int ms) {
    CHECK(co::scheduler()) << "must be called in coroutine..";
    int r, e;
    int fd = SSL_get_fd(s);
    if (fd <0) return -1;

    do {
        ERR_clear_error();
        r = SSL_read(s, buf, n);
        if (r> 0) return r; // success
        if (r == 0) {
            DLOG << "SSL_read return 0, error: "<< SSL_get_error(s, 0);
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
            DLOG << "SSL_read return "<< r << ", error:" << e;
            return r;
        }
    } while (true);
}
```


The above is an example of coroutineizing SSL_read in openssl. The whole process is relatively simple. The underlying socket must be non-blocking. When SSL_read generates a `SSL_ERROR_WANT_READ` error, use co::IoEvent to wait for the read event, and when an `SSL_ERROR_WANT_WRITE` error occurs, use co::IoEvent to wait for the write event. When wait() returns normally, it means that the socket is readable or writable, continue to call SSL_read to complete the I/O operation. 


At present, co has successfully coroutineized openssl and libcurl. In theory, all third-party network libraries that support non-blocking I/O can be coroutineized in a similar way to the above.




### System API hook


API hook is simply to intercept the system API call. If it is found that the call is in a coroutine and a blocking socket is used, the socket is modified to non-blocking mode, and then co::IoEvent or the lower-level interface in co is used to wait for the I/O event on the socket. When an I/O event arrives, wake up the coroutine to continue the I/O operation. 


Since co 2.0.1, hook has been supported on linux/mac/windows platforms.


The advantage of API hook is that, we only hook a small number of system socket APIs, and we can use all third-party libraries that provide blocking APIs in coroutine. 


However, coroutineization needs to provide a set of coroutineized APIs for each third-party library, but it has better performance than API hook, and is safer, and can avoid some problems caused by the complexity of the third-party library. 




