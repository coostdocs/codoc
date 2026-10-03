---
weight: 2
title: "Memory Allocation"
---

## Header

```cpp
#include "co/mem.h"
```

The API is in the `co` namespace.

{{< hint warning >}}
APIs starting with an underscore are generally used internally by coost and are not recommended for users to call.
{{< /hint >}}

## Overview

The coost memory allocator divides memory into three categories:

| Category | Size Range | Alignment |
| ---- | --------------- | ------------------------------ |
| Small memory | `<= S` (S is less than 4K) | 16-byte aligned |
| Medium memory | `<= 128K` | 4K aligned |
| Large memory | `> 128K` | Page-aligned, directly `mmap` / `VirtualAlloc` |

{{< hint warning >}}
Unlike ordinary memory allocators, the coost allocator does not store the size of allocated memory; free requires passing the size.
{{< /hint >}}

This design has the following benefits:
- The allocator does not need to write metadata at the beginning of memory; the memory layout obtained by users is compact, with no extra overhead;
- During free, the category can be determined directly from the size without reading metadata, making the path simple;
- It is cache-friendly, and allocation and deallocation are faster.

## Basic Allocation Functions

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

- The above APIs are all thread-safe.
- In `co::free(p, n)`, `n` must be the same as at allocation time; otherwise, undefined behavior may result.
- `co::realloc(p, o, n)` requires `n > o`, i.e. it can only expand; on failure it returns `nullptr`, and the original `p` is unaffected and remains valid.
- `zalloc` allocates memory and zeroes it.
- `alloc` and `zalloc` support allocating memory aligned to `align`; `align` must be a power of 2, and the maximum allowed value is 256.
- `valloc` **allocates page-aligned memory with contents zeroed**; in `vfree`, `n` must be the same as at allocation time.
- `strdup` creates a string copy; when releasing it, use `co::free(s, strlen(s) + 1)`:

{{< hint warning >}}
Memory allocated by valloc does not support realloc; it is usually used to allocate large blocks of memory that do not need realloc.
{{< /hint >}}

## Static Object Construction

```cpp
// make static object, which will be destructed automatically at exit
template<typename T, typename... Args>
inline T* co::make_static(Args&&... args);

// make non-dependent static object at the root level
template<typename T, typename... Args>
inline T* co::make_rootic(Args&&... args);
```

- Creates a static object; the returned pointer **must not be manually freed or deleted by the user**.
- For objects created by `co::make_static`, coost destroys them at program exit in the order of first constructed, later destroyed.
- Objects created by `co::make_rootic` are always destroyed last; it is generally used to create dependency-free static objects.

Example:

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

    // Cross-type conversion: requires T to be a base class of X, and T has a virtual destructor
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

- Similar to `std::unique_ptr`.
- Only move semantics are supported, ensuring the object is always managed by a unique `unique`.

Example:

```cpp
co::unique<co::string> s = co::make_unique<co::string>(32, 'x');
co::println("*s = ", *s);

// move semantics, s -> nullptr
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

    // Cross-type conversion: requires T to be a base class of X, and T has a virtual destructor
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

- Similar to `std::shared_ptr`.
- If the internally held pointer is `nullptr`, copying does not increase the reference count.

Example:

```cpp
co::shared<co::string> s = co::make_shared<co::string>(32, 'x');
co::println("use_count = ", s.use_count()); // 1

// Copy, increases the reference count
co::shared<co::string> t = s;
co::println("use_count = ", s.use_count()); // 2

// An empty object has an internal pointer of nullptr; copying does not create sharing
co::shared<int> x;
co::shared<int> y;
y = x;

// *x == 7, y == nullptr
x = co::make_shared<int>(7);
co::println(x.use_count()); // 1
co::println(y.use_count()); // 0
```

## unique / shared Construction Constraints

`co::unique<T>` and `co::shared<T>` **are not allowed** to be constructed directly from dynamically allocated memory; you can only use:

```cpp
co::make_unique<T>(args...);
co::make_shared<T>(args...);
```

## co::stl_allocator

Used to replace `std::allocator` in STL containers.

`co/stl.h` provides commonly used STL containers with the memory allocator already replaced by `co::stl_allocator`.

Example:

```cpp
std::vector<int, co::stl_allocator<int>> v;
v.push_back(1);
v.push_back(2);

co::vecotr<int> x;
x.push_back(8);
co::println("x: ", x);
```

## Notes

- The coost allocator **cannot replace** `malloc, free` or `operator new, operator delete`.
- `co::free` requires the memory size, and `co::realloc` requires the old size, which differs from standard library semantics.
- Memory allocated by `malloc` or `new` cannot be freed with `co::free`, and vice versa.
