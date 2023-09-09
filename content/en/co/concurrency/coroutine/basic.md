---
weight: 1
title: "Basic concepts"
---


## Basic concepts of coroutine

- Coroutines are lightweight scheduling units that run in threads.
- Coroutines are to threads, similar to threads to processes.
- There can be multiple threads in a process and multiple coroutines in a thread.
- The thread where the coroutine runs in is generally called the **scheduling thread**.
- The scheduling thread will suspend a coroutine, if it blocks on an I/O operation or sleep was called in the coroutine.
- When a coroutine is suspended, the scheduling thread will switch to other coroutines waiting to be executed.
- Switching of coroutines is done in user mode, which is faster than switching between threads.


Coroutines are very suitable for network programming, and can achieve synchronous programming without asynchronous callbacks, which greatly reduces the programmer's mental burden. 


Coost implements a [go-style](https://github.com/golang/go/) coroutine with the following features:

- Support multi-thread scheduling, the default number of threads is the number of system CPU cores.
- Coroutines in the same thread share several stacks (the default size is 1MB), and the memory usage is low. Test on Linux shows that 10 millions of coroutines only take 2.8G of memory (for reference only).
- Once a coroutine is created, it always runs in the same thread.
- There is a flat relationship between coroutines, and new coroutines can be created from anywhere (including in coroutines).


The relevant code for context switching is taken from [tbox](https://github.com/tboox/tbox/) by [ruki](https://github.com/waruqi), and tbox refers to the implementation of [boost](https://www.boost.org/doc/libs/1_70_0/libs/context/doc/html/index.html), thanks here!


