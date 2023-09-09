---
weight: 8
title: "Coroutine Pool"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::pool

`co::pool` is a general-purpose coroutine pool, which is **coroutine-safe** and internally stores `void*` type pointers, which can be used as connection pool, memory pool or caches for other purposes.


### constructor

```cpp
1. pool();
2. pool(pool&& p);
3. pool(const pool& p);
4. pool(std::function<void*()>&& ccb, ​​std::function<void(void*)>&& dcb, size_t cap=(size_t)-1);
```

- 1, default constructor, compared to 4, `ccb` and `dcb` are NULL.
- 2, move constructor.
- 3, copy constructor, only increases the internal reference count by 1.
- 4, `ccb` is used to create an element, and `dcb` is used to destroy an element, `cap` specifies the maximum capacity of the pool, the default is -1, unlimited capacity.
- Note that the parameter `cap` is not the total capacity. It is for a single thread. In the internal implementation of `co::pool`, each thread has its own pool. If the cap is set to 1024 and there are 8 scheduling threads, then the total capacity is 8192.
- When `dcb` is NULL, parameter `cap` will be ignored, because `co::pool` needs to use `dcb` to destroy extra elements when the number of elements exceeds the maximum capacity.


- Example

```cpp
class T {};
co::pool p(
     []() { return (void*) new T; }, // ccb
     [](void* p) { delete (T*) p; }, // dcb
);
```



### clear

```cpp
void clear() const;
```

- Clear the pool, can be called anywhere.
- If `dcb` is set, elements in the pool will be destroyed with `dcb`.



### pop

```cpp
void* pop() const;
```

- Pop an element from the pool, **Must be called in coroutine**.
- When the pool is empty and `ccb` is not NULL, `ccb()` will be called to create an element and return, otherwise return NULL.
- This method is coroutine safe and does not require locking.



### push

```cpp
void push(void* e) const;
```

- Put the element back to the pool, **must be called in coroutine**.
- When parameter `e` is NULL, it is ignored.
- Since each thread has its own pool internally, **push() and pop() methods need to be called in the same thread**.
- If the pool has reached the maximum capacity and `dcb` is not NULL, `dcb(e)` will be called to destroy the element.
- This method is coroutine safe and does not require locking.



### size

```cpp
size_t size() const;
```

- Returns the pool size of the current thread, **must be called in the coroutine**.




## co::pool_guard

`co::pool_guard` automatically pops an element from `co::pool` when it is constructed, and automatically puts the element back when it is destructed.

```cpp
template<typename T>
class pool_guard;
```

- Parameter `T` is the actual type pointed to by the pointer in `co::pool`.



### constructor

```cpp
explicit pool_guard(co::pool& p);
explicit pool_guard(co::pool* p);
```

- Pop an element from `co::pool`, the parameter `p` is a reference or pointer to the `co::pool` class.



### destructor

```cpp
~pool_guard();
```

- Push the element obtained in the constructor back to `co::pool`.



### get

```cpp
T* get() const;
```

- Get the pointer taken from `co::pool`.



### operator->

```cpp
T* operator->() const;
```

- Returns the pointer taken from `co::pool`.



### operator*

```cpp
T& operator*() const;
```

- Returns a reference to the object pointed to by the internal pointer.



### operator==

```cpp
bool operator==(T* p) const;
```

- Determine whether the internal pointer is equal to `p`.



### operator!=

```cpp
bool operator!=(T* p) const;
```

- Determine whether the internal pointer is not equal to `p`.



### operator bool

```cpp
explicit operator bool() const;
```

- Returns true if the internal pointer is not NULL, false otherwise.



### Code Example

```cpp
class Redis; // assume class Redis is a connection to the redis server

co::pool p(
     []() { return (void*) new Redis; }, // ccb
     [](void* p) { delete (Redis*) p; }, // dcb
     8192 //cap
);

void f() {
     co::pool_guard<Redis> rds(p);
     rds->get("xx");
}

go(f);
```
