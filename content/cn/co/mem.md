---
weight: 2
title: "内存分配"
---

## 头文件

```cpp
#include "co/mem.h"
```

API 在 `co` 命名空间。

`_` 开头的 API 一般是 coost 内部使用的，不建议用户调用。

内部通过静态对象（nifty counter）初始化，只会初始化一次，用户不需要显式初始化。


## 概述

coost 内存分配器与 glibc 不同：**内部不保存所分配内存的大小**，因此释放时需要用户传入大小。

分配器把内存按大小分为三个等级：

| 等级   | 大小范围            | 对齐                             |
| ---- | --------------- | ------------------------------ |
| 小内存  | `<= S`（S 小于 4K） | 16 字节对齐                        |
| 中等内存 | `<= 128K`       | 4K 对齐                          |
| 大内存  | `> 128K`        | 页对齐，直接 `mmap` / `VirtualAlloc` |

说明：

- `co::free(p, n)` 根据 `n` 判断内存属于哪一类，再回收。
- 因此 `n` 要求与 `alloc` / `realloc` 时传入的 size 一致。
- 传错 `n` 可能导致未定义行为。
- coost 分配的内存至少是 16 字节对齐。


## 基础分配函数

```cpp
void* co::alloc(size_t n);

// @align: must be power of 2, and its maximum value is 256
void* co::alloc(size_t n, size_t align);

// @p: may be NULL
// @n: MUST be the same as the size used in alloc or realloc
void co::free(void* p, size_t n);

// @p: may be NULL
// @o: old size, must be the same as the size used in alloc() or a previous realloc()
// @n: new size, must be greater than @o
// return: may be the same as @p, or NULL on failure
void* co::realloc(void* p, size_t o, size_t n);

// alloc and zero-clear
void* co::zalloc(size_t n);
void* co::zalloc(size_t n, size_t align);

// virtual alloc, page-aligned and zero-cleared
void* co::valloc(size_t n);

// virtual free
void co::vfree(void* p, size_t n);

char* co::strdup(const char* s);
```

### 关键约束

- 上述 API 都是线程安全的。
- `co::free(p, n)` 的 `n` 必须与分配时一致，否则未定义行为。
- `co::realloc(p, o, n)` 要求 `n > o`，即只能扩张。
  - 实际应用中 `realloc` 一般也只用于扩张。
  - 失败时返回 `nullptr`，原来的 `p` 不受影响，仍然有效。
- `align` 必须是 2 的幂，最大值是 256 保证。
- `valloc` / `vfree`：
  - 分配按页对齐的内存，内容清零；
  - `n` 不必是页大小的整数倍，系统 API 会自动 round up；
  - 释放时 `n` 必须与分配时一致；
  - 一般用于不需要 `realloc` 的大块内存。
- `strdup` 返回的内存由 coost 分配器管理，释放时用 `co::free(s, strlen(s) + 1)`：



## 静态对象构造

```cpp
// make static object, which will be destructed automatically at exit
//   - T* p = co::make_static<T>(args)
template<typename T, typename... Args>
inline T* co::make_static(Args&&... args);

// make non-dependent static object at the root level
template<typename T, typename... Args>
inline T* co::make_rootic(Args&&... args);
```

说明：

- 创建静态对象，返回的指针由 coost 管理，程序退出时自动析构，**用户不需要、也不能手动 free 或 delete**。
- `make_rootic` 慎用，它创建的静态对象总是在最后析构，一般用于创建无依赖的静态对象。


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

说明：

- 类似 `std::unique_ptr`。
- 只支持移动语义，保证 `unique` 中的对象始终由唯一一个 `unique` 对象管理。

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

说明：

- 与 `std::shared_ptr` 类似。
- 若内部持有的指针为 `nullptr`，拷贝并不会增加引用计数。


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

## unique / shared 的构造约束

`co::unique<T>` 和 `co::shared<T>` **不允许**从动态分配的内存直接构造，只能使用：

```cpp
co::make_unique<T>(args...);
co::make_shared<T>(args...);
```

## co::stl_allocator

用于替换 STL 容器中的 `std::allocator`。

`co/stl.h` 提供常用的 STL 容器，内存分配器已替换为 `co::stl_allocator`。

示例：

```cpp
std::vector<int, co::stl_allocator<int>> v;
v.push_back(1);
v.push_back(2);

#include "co/stl.h"

co::vecotr<int> x;
x.push_back(8);
co::println("x: ", x);
```



## 与标准库的关系

- coost 分配器**不能替代** `malloc, free` 或 `operator new, operator delete`。
- `co::free` 需要带内存大小，`co::realloc` 需要带 old size，与标准库语义不同。
- 用 `malloc` 或 `new` 分配的内存不能用 `co::free` 释放，反之亦然。
- `co::stl_allocator` 与 `std::allocator` 接口兼容，但底层使用 coost 分配器。



## 注意事项

- `co::free(p, n)` 的 `n` 必须与分配时一致，否则未定义行为。
- `co::realloc(p, o, n)` 要求 `n > o`；失败时返回 `nullptr`，原 `p` 仍有效。
- `co::alloc(n, align)` 的 `align` 必须是 2 的幂，最大 256。
- `co::strdup` 返回的内存需用 `co::free(s, strlen(s) + 1)` 释放。
- `co::unique<T>` 只支持移动语义。
- `malloc` 或 `new` 分配的内存不能用 `co::free` 释放。
