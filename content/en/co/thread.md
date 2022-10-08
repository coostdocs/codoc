---
weight: 10
title: "Thread"
---

include: [co/thread.h](https://github.com/idealvin/coost/blob/master/include/co/thread.h).


## Thread


The `Thread` class is an encapsulation of the thread. When a Thread object is created, the thread will be started. Once the thread is started, it will run until the thread function exits. The Thread class does not provide a method for forcing terminating the thread (something like pthread_cancel). This type of method is usually unsafe and may cause damage to user data. 




### Thread::Thread


```cpp
explicit Thread(co::Closure* cb);

template<typename F>
explicit Thread(F&& f);

template<typename F, typename P>
Thread(F&& f, P&& p);

template<typename F, typename T, typename P>
Thread(F&& f, T* t, P&& p);
```


- Constructor, the thread will start after the Thread object is created.
- In the first version, the parameter cb is a pointer of Closure type. Generally, users are not recommended to use this version directly.
- For the 2-4th version, a Closure object is constructed according to the passed in parameters, and then the first version is called.
- In the second version, the parameter f is any runnable object, as long as we can call `f()` or `(*f)()`.
- In the third version, the parameter f is any runnable object, as long as we can call `f(p)`, `(*f)(p)` or `(p->*f)()`.
- In the fourth version, the parameter f is a method with one parameter in class T, the parameter t is a pointer of type `T`, and p is the parameter of f.



- Example



```cpp
Thread x([](){});                  // lambda
Thread x(f);                       // void f();
Thread x(f, p);                    // void f(void*); void* p;
Thread x(f, 7);                    // void f(int v);
Thread x(&T::f, &t);               // void T::f(); T t;
Thread x(&T::f, &t, 7);            // void T::f(int v); T t;
Thread x(std::bind(&T::f, &t, 7)); // void T::f(int v); T t;
```




### Thread::~Thread


```cpp
Thread::~Thread();
```


- Destructor, call the `join()` method, wait for the thread to exit, and release system resources.





### Thread::detach


```cpp
void detach();
```


- Let the thread run independently of the Thread object. Once this method is called, the Thread object is useless. When the thread function exits, the system resources are automatically released.



- Example



```cpp
void f();
Thread(f).detach(); // run f() in a thread
```




### Thread::join


```cpp
void join();
```


- Calling this method will block until the thread function exits, and then release system resources.
- If the `detach()` method has been called before, nothing will happen when calling this method.






## co::thread_id
```cpp
uint32 thread_id();
```

- This function returns the id of the current thread.
- In the internal implementation, [TLS](https://wiki.osdev.org/Thread_Local_Storage) is used to save the thread id, and each thread only needs one system call.





## current_thread_id


```cpp
uint32 current_thread_id();
```

- Deprecated since v2.0.2, use `co::thread_id()` instead.





## Mutex (Mutex)


`Mutex` is a kind of mutual exclusion lock commonly used in multithreaded programming. At the same time, at most one thread holds the lock, and other threads must wait for the lock to be released. 




### Mutex::Mutex


```cpp
Mutex();
```


- Constructor, allocate system resources and initialize.





### Mutex::~Mutex


```cpp
Mutex::~Mutex();
```


- Destructor, release system resources.





### Mutex::lock


```cpp
void lock();
```


- Acquire the lock, the method will block until the lock is successfully acquired.





### Mutex::try_lock


```cpp
bool try_lock();
```


- Acquire the lock, will not block. If the lock is successfully acquired, it returns true, otherwise it returns false.





### Mutex::unlock


```cpp
void unlock();
```


- Release the lock, this method is generally called by the thread that held the lock.







## MutexGuard


The `MutexGuard` class is used to automatically acquire and release the lock in `Mutex` to prevent users from forgetting to release the lock. 




### MutexGuard::MutexGuard


```cpp
explicit MutexGuard(Mutex& m);
explicit MutexGuard(Mutex* m);
```


- The constructor, the parameter m is a reference or pointer of the `Mutex`. When it is a pointer, m MUST NOT be NULL.
- Internally, m's `lock()` method is called to acquire the lock.





### MutexGuard::~MutexGuard


```cpp
MutexGuard::~MutexGuard();
```


- Destructor, release the lock acquired in the constructor.



- Example



```cpp
Mutex m;
MutexGuard g(m);
```






## SyncEvent


`SyncEvent` is a synchronization mechanism between multiple threads, suitable for the producer-consumer model. The consumer calls the `wait()` method to wait for the synchronization signal, and the producer calls the `signal()` method to generate the synchronization signal. SyncEvent supports multiple producers and multiple consumers, but in actual applications, there is usually a single consumer. 




### SyncEvent::SyncEvent


```cpp
explicit SyncEvent(bool manual_reset = false, bool signaled = false);
```


- Constructor, the parameter manual_reset indicates whether to manually set the synchronization state to unsynchronized, and the parameter signaled indicates whether the initial state is synchronized.





### SyncEvent::~SyncEvent


```cpp
SyncEvent::~SyncEvent();
```


- Destructor, release system resources.





### SyncEvent::reset


```cpp
void reset();
```


- This method sets SyncEvent to unsynchronized state.
- When manual_reset is true in the constructor, the user needs to manually call this method after wait() returns, otherwise the SyncEvent may remain in a synchronized state forever.





### SyncEvent::signal


```cpp
void signal();
```


- This method generates a synchronization signal and sets SyncEvent to a synchronized state.





### SyncEvent::wait


```cpp
void wait();
bool wait(uint32 ms);
```


- The first version will wait until SyncEvent becomes synchronized.
- The second version will wait until SyncEvent becomes synchronized or timed out. The parameter ms specifies the timeout period in milliseconds.
- The second version returns true when the SyncEvent becomes synchronized, and returns false when it timed out.
- When manual_reset is false in the constructor, SyncEvent will be automatically set to unsynchronized when wait() ends.





### Code example


```cpp
bool manual_reset = false;
SyncEvent ev(manual_reset);

void f1() {
    if (!ev.wait(1000)) {
        LOG << "f1: timedout..";
    } else {
        LOG << "f1: event signaled..";
        if (manual_reset) ev.reset();
    }
}

void f2() {
    LOG << "f2: send a signal..";
    ev.signal();
}

Thread(f1).detach();
Thread(f2).detach();
```






## TLS-based thread_ptr


```cpp
template <typename T, typename D=std::default_delete<T>>
class thread_ptr;
```


The `thread_ptr` class is similar to `std::unique_ptr`, but uses the [TLS](https://wiki.osdev.org/Thread_Local_Storage) mechanism internally, and each thread sets and owns its own ptr. 




### thread_ptr::thread_ptr


```cpp
thread_ptr();
```


- Constructor, allocate system resources and initialize.





### thread_ptr::~thread_ptr


```cpp
thread_ptr::~thread_ptr();
```


- Destructor, delete the private ptr of each thread, and release TLS related system resources.





### thread_ptr::get


```cpp
T* get() const;
```


- Return the ptr of the current thread.
- If the thread has not set a ptr before, this method returns NULL.





### thread_ptr::operator=


```cpp
void operator=(T* p);
```


- Assignment, set the current thread's ptr to p, which is equivalent to `reset(p)`.





### thread_ptr::operator->


```cpp
T* operator->() const;
```


- Overload `operator->`, returns the ptr of the current thread.





### thread_ptr::operator*


```cpp
T& operator*() const;
```


- Overload `operator*`, returns the reference of the object pointed to by the ptr of the current thread.





### thread_ptr::operator==


```cpp
bool operator==(T* p) const;
```


- Determine whether the current thread's ptr is equal to p.





### thread_ptr::operator!=


```cpp
bool operator!=(T* p) const;
```


- Determine whether the current thread's ptr is not equal to p.





### thread_ptr::operator!


```cpp
bool operator!() const;
```


- Determine whether the ptr of the current thread is NULL, return true if it is NULL, otherwise return false.





### thread_ptr::operator bool


```cpp
explicit operator bool() const;
```


- Convert thread_ptr to bool type. If the internal pointer is not NULL, return true, otherwise return false.





### thread_ptr::release


```cpp
T* release();
```


- Release the ptr of the current thread.
- This method sets the ptr of the current thread to NULL and returns the previous ptr.





### thread_ptr::reset


```cpp
void reset(T* p = 0);
```


- Reset the ptr of the current thread to p, p is 0 by default, and the previous ptr will be deleted by `D()(x)`.





### Code example


```cpp
struct T {
    T(int v): _v(v) {}
    void run() {
        LOG << current_thread_id() << "v:" << _v;
    }
    int _v;
};

thread_ptr<T> pt;

// Each thread will set its own pointer, and different threads will not affect each other
void f(int v) {
    if (pt == NULL) {
        LOG << "new T(" << v << ")";
        pt.reset(new T(v));
    }
    pt->run();
}

Thread(f, 1).detach(); // start thread 1
Thread(f, 2).detach(); // start thread 2
```
