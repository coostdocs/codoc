---
weight: 2
title: "内存分配"
---

## 头文件

```cpp
#include "co/mem.h"
```

API 在 `co` 命名空间。

{{< hint warning >}}
下划线开头的 API 一般 coost 内部使用，不建议用户调用。
{{< /hint >}}


## 概述

coost 内存分配器将内存分为三类:

| 类别   | 大小范围            | 对齐                             |
| ---- | --------------- | ------------------------------ |
| 小内存  | `<= S`（S 小于 4K） | 16 字节对齐                        |
| 中等内存 | `<= 128K`         | 4K 对齐                          |
| 大内存  | `> 128K`           | 页对齐，直接 `mmap` / `VirtualAlloc` |

{{< hint warning >}}
与一般内存分配器不同，coost 分配器不保存分配内存的大小，free 时需要传大小。
{{< /hint >}}

这种设计有如下好处：
- 分配器不需要在内存头部写元数据，用户拿到的内存布局紧凑，没有额外开销；
- free 时根据大小直接判断属于哪一类别，不需要读元数据，路径简单；
- 缓存友好，分配和释放更快。


## 基础分配函数

```cpp
void* co::alloc(size_t n);

// @align: must be power of 2, and its maximum value is 256
void* co::alloc(size_t n, size_t align);

// @p: may be NULL
// @n: MUST be the same as the size used in alloc or realloc
void co::free(void* p, size_t n);

// @p: may be NULL
// @o: old size, must be the same as the size used in alloc or realloc
// @n: new size, must be greater than @o
// return: may be the same as @p, or NULL on failure
void* co::realloc(void* p, size_t o, size_t n);

// alloc and zero-clear
void* co::zalloc(size_t n);
void* co::zalloc(size_t n, size_t align);

// virtual alloc, page-aligned and zero-cleared
void* co::valloc(size_t n);
void co::vfree(void* p, size_t n);

char* co::strdup(const char* s);
```

- 上述 API 均线程安全。
- `co::free(p, n)` 中 `n` 必须与分配时一致，否则可能导致未定义行为。
- `co::realloc(p, o, n)` 要求 `n > o`，即只能扩张；失败时返回 `nullptr`，原 `p` 不受影响，仍然有效。
- `zalloc` 分配内存并清零。
- `alloc`, `zalloc` 支持分配按 `align` 对齐的内存，`align` 必须是 2 的幂，最大允许值是 256。
- `valloc` **分配按页对齐内存，内容清零**, `vfree` 中 `n` 必须与分配时一致。
- `strdup` 创建字符串副本，释放时用 `co::free(s, strlen(s) + 1)`：

{{< hint warning >}}
valloc 分配的内存不支持 realloc，通常用于分配不需要 realloc 的大块内存。
{{< /hint >}}


## 静态对象构造

```cpp
// make static object, which will be destructed automatically at exit
template<typename T, typename... Args>
inline T* co::make_static(Args&&... args);

// make non-dependent static object at the root level
template<typename T, typename... Args>
inline T* co::make_rootic(Args&&... args);
```

- 创建静态对象，返回的指针，**用户不能手动 free 或 delete**。
- `co::make_static` 创建的对象，coost 在程序退出时按先构造、后析构的顺序析构。
- `co::make_rootic` 创建的对象总是在最后析构，一般用于创建无依赖的静态对象。

示例：

```cpp
co::string* g_s = co::make_static<co::string>(32, 'x');
```


## co::unique

```cpp
template<typename T>
struct unique {
    constexpr unique() noexcept;
    constexpr unique(std::nullptr_t) noexcept;
    unique(unique& x) noexcept;
    unique(unique&& x) noexcept;
    ~unique();

    unique(const unique&) = delete;

    unique& operator=(unique&& x) noexcept;
    unique& operator=(unique& x) noexcept;

    // 跨类型转换：要求 T 是 X 的基类，且 T 有虚析构函数
    template<typename X> unique(unique<X>& x) noexcept;
    template<typename X> unique(unique<X>&& x) noexcept;
    template<typename X> unique& operator=(unique<X>&& x) noexcept;
    template<typename X> unique& operator=(unique<X>& x) noexcept;

    T* get() const noexcept;
    T* operator->() const noexcept;   // runtime_assert(_p)
    T& operator*() const noexcept;    // runtime_assert(_p)

    bool operator==(T* p) const noexcept;
    bool operator!=(T* p) const noexcept;
    explicit operator bool() const noexcept;

    void reset() noexcept;
    void swap(unique& x) noexcept;
    void swap(unique&& x) noexcept;

    union { T* _p; uint32* _s; };
};

template<typename T, typename... Args>
inline unique<T> co::make_unique(Args&&... args);
```

- 类似 `std::unique_ptr`。
- 只支持移动语义，保证对象始终由唯一 `unique` 管理。

示例：

```cpp
co::unique<co::string> s = co::make_unique<co::string>(32, 'x');
co::println("*s = ", *s);

// move 语义，s -> nullptr
co::unique<co::string> x = s;
```


## co::shared

```cpp
template<typename T>
struct shared {
    constexpr shared() noexcept;
    constexpr shared(std::nullptr_t) noexcept;

    shared(const shared& x) noexcept;
    shared(shared&& x) noexcept;
    ~shared();

    shared& operator=(const shared& x) noexcept;
    shared& operator=(shared&& x) noexcept;

    // 跨类型转换：要求 T 是 X 的基类，且 T 有虚析构函数
    template<typename X> shared(const shared<X>& x) noexcept;
    template<typename X> shared(shared<X>&& x) noexcept;
    template<typename X> shared& operator=(const shared<X>& x) noexcept;
    template<typename X> shared& operator=(shared<X>&& x) noexcept;

    T* get() const noexcept;
    T* operator->() const noexcept;
    T& operator*() const noexcept;

    bool operator==(T* p) const noexcept;
    bool operator!=(T* p) const noexcept;
    explicit operator bool() const noexcept;

    void reset() noexcept;
    size_t ref_count() const noexcept;
    size_t use_count() const noexcept;
    void swap(shared& x) noexcept;
    void swap(shared&& x) noexcept;

    union { T* _p; uint32* _s; };
};

template<typename T, typename... Args>
inline shared<T> co::make_shared(Args&&... args);
```

- 类似 `std::shared_ptr`。
- 若内部持有指针为 `nullptr`，拷贝不会增加引用计数。

示例：

```cpp
co::shared<co::string> s = co::make_shared<co::string>(32, 'x');
co::println("use_count = ", s.use_count()); // 1

// 拷贝，增加引用计数
co::shared<co::string> t = s;
co::println("use_count = ", s.use_count()); // 2

// 空对象内部指针为 nullptr, 拷贝不形成共享
co::shared<int> x;
co::shared<int> y;
y = x;

// *x == 7, y == nullptr
x = co::make_shared<int>(7);
co::println(x.use_count()); // 1
co::println(y.use_count()); // 0
```


## unique / shared 构造约束

`co::unique<T>` 和 `co::shared<T>` **不允许**从动态分配的内存直接构造，只能使用：

```cpp
co::make_unique<T>(args...);
co::make_shared<T>(args...);
```


## co::stl_allocator

用于替换 STL 容器中的 `std::allocator`。

`co/stl.h` 提供常用 STL 容器，内存分配器已替换为 `co::stl_allocator`。

示例：

```cpp
std::vector<int, co::stl_allocator<int>> v;
v.push_back(1);
v.push_back(2);

co::vecotr<int> x;
x.push_back(8);
co::println("x: ", x);
```


## 注意事项

- coost 分配器**不能替代** `malloc, free` 或 `operator new, operator delete`。
- `co::free` 需要带内存大小，`co::realloc` 需要带 old size，与标准库语义不同。
- `malloc` 或 `new` 分配的内存不能用 `co::free` 释放，反之亦然。
