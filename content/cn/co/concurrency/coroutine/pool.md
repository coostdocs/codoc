---
weight: 8
title: "协程池"
---

include: [co/co.h](https://github.com/idealvin/coost/blob/master/include/co/co.h).


## co::pool

`co::pool` 是一种通用的协程池，它是**协程安全**的，内部存储 `void*` 类型的指针，可以用作连接池、内存池或其他用途的缓存。


### constructor

```cpp
1. pool();
2. pool(pool&& p);
3. pool(const pool& p);
4. pool(std::function<void*()>&& ccb, std::function<void(void*)>&& dcb, size_t cap=(size_t)-1);
```

- 1, 默认构造函数，与 4 相比，`ccb` 与 `dcb` 为 NULL。
- 2, 移动构造函数。
- 3, 拷贝构造函数，仅将内部引用计数加 1。
- 4, 参数 `ccb` 用于创建元素，参数 `dcb` 用于销毁元素，参数 `cap` 指定 pool 的最大容量，默认为 -1，不限容量。
- 注意参数 cap 并不是总容量，它是对单个线程而言，在 `co::pool` 内部实现中，每个线程都有自己的 pool，如 cap 设置为 1024，调度线程有 8 个，则总容量是 8192。
- 当 `dcb` 为 NULL 时，参数 `cap` 会被忽略，这是因为当元素个数超过最大容量时，`co::pool` 需要用 `dcb` 销毁多余的元素。


- 示例

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

- 清空 pool，可以在任何地方调用。
- 如果设置了 `dcb`，会用 `dcb` 销毁 pool 中的元素。



### pop

```cpp
void* pop() const;
```

- 从 pool 中取出一个元素，**必须在协程中调用**。
- pool 为空时，若 `ccb` 不是 NULL，则调用 `ccb()` 创建一个元素并返回，否则返回 NULL。
- 此方法是协程安全的，不需要加锁。



### push

```cpp
void push(void* e) const;
```

- 将元素放回 pool 中，**必须在协程中调用**。
- 参数 e 为 NULL 时，直接忽略。
- 由于每个线程在内部拥有自己的 pool，**push() 与 pop() 方法需要在同一个线程中调用**。
- 若 pool 已经达到最大容量，且 dcb 不为 NULL，则直接调用 `dcb(e)` 销毁该元素。
- 此方法是协程安全的，不需要加锁。



### size

```cpp
size_t size() const;
```

- 返回当前线程的 pool 大小，**必须在协程中调用**。




## co::pool_guard

`co::pool_guard` 在构造时自动从 `co::pool` 取出元素，析构时自动将元素放回 `co::pool`。同时，它还重载了 `operator->`，可以像智能指针一样使用它。

```cpp
template<typename T>
class pool_guard;
```

- 参数 T 是 `co::pool` 中指针所指向的实际类型。



### constructor

```cpp
explicit pool_guard(co::pool& p);
explicit pool_guard(co::pool* p);
```

- 从 `co::pool` 中取出一个元素，参数 `p` 是 `co::pool` 类的引用或指针。



### destructor

```cpp
~pool_guard();
```

- 将构造函数中获取的元素，放回 `co::pool` 中。



### get

```cpp
T* get() const;
```

- 获取从 `co::pool` 中取出的指针。



### operator->

```cpp
T* operator->() const;
```

- 返回从 `co::pool` 中取出的指针。



### operator*

```cpp
T& operator*() const;
```

- 返回内部指针所指向对象的引用。



### operator==

```cpp
bool operator==(T* p) const;
```

- 判断内部指针是否等于 p。



### operator!=

```cpp
bool operator!=(T* p) const;
```

- 判断内部指针是否不等于 p。



### operator bool

```cpp
explicit operator bool() const;
```

- 若内部指针不是 NULL，返回 true，否则返回 false。



### 代码示例

```cpp
class Redis;  // assume class Redis is a connection to the redis server

co::pool p(
    []() { return (void*) new Redis; }, // ccb
    [](void* p) { delete (Redis*) p; }, // dcb
    8192                                // cap
);

void f() {
    co::pool_guard<Redis> rds(p);
    rds->get("xx");
}

go(f);
```

上面的例子相当于 redis 连接池。如果使用 CLS 机制，一个协程一个连接，则 100 万协程需要建立 100 万连接，消耗较大。但使用 pool 机制，100 万协程可能只需要共用少量的连接。pool 机制比 CLS 更高效、更合理，这是 coost 不支持 CLS 的原因。

