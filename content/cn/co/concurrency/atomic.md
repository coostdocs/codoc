---
weight: 1
title: "原子操作"
---


## 头文件

```cpp
#include "co/atomic.h"
```

API 在 `co` 命名空间。


## 内存序

```cpp
using memorder_t = std::memory_order;
constexpr memorder_t mo_relaxed = std::memory_order_relaxed;
constexpr memorder_t mo_consume = std::memory_order_consume;
constexpr memorder_t mo_acquire = std::memory_order_acquire;
constexpr memorder_t mo_release = std::memory_order_release;
constexpr memorder_t mo_acq_rel = std::memory_order_acq_rel;
constexpr memorder_t mo_seq_cst = std::memory_order_seq_cst;
```


## 加载与存储

```cpp
template<typename T>
T atomic_load(const T* p, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
void atomic_store(T* p, V v, memorder_t mo = mo_seq_cst);
```

- `atomic_load` 支持：`mo_relaxed`、`mo_consume`、`mo_acquire`、`mo_seq_cst`。
- `atomic_store` 支持：`mo_relaxed`、`mo_release`、`mo_seq_cst`。

示例:

```cpp
int i = 0;
co::atomic_store(&i, 3);      // i -> 3
int x = co::atomic_load(&i);  // x -> 3
```


## 交换

```cpp
template<typename T, typename V>
T atomic_swap(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename O, typename V>
T atomic_compare_swap(
    T* p, O o, V v,
    memorder_t smo = mo_seq_cst, memorder_t fmo = mo_seq_cst
);

template<typename T, typename O, typename V>
T atomic_cas(
    T* p, O o, V v,
    memorder_t smo = mo_seq_cst, memorder_t fmo = mo_seq_cst
);

template<typename T, typename O, typename V>
bool atomic_bool_cas(
    T* p, O o, V v,
    memorder_t smo = mo_seq_cst, memorder_t fmo = mo_seq_cst
);
```

- `atomic_swap` 返回旧值，支持所有内存序。
- `atomic_cas` 与 `atomic_compare_swap` 等价，`*p == o` 时才执行交换操作，返回旧值。
- `atomic_bool_cas` 与 `atomic_cas` 类似，成功返回 true，失败返回 false。
- CAS 操作中，`smo` 是成功时内存序，支持所有，`fmo` 是失败时内存序，不能是 `mo_release`、`mo_acq_rel`，且不能强于 `smo`。

示例:

```cpp
bool b = false;
bool o = co::atomic_swap(&b, true); // b -> true, o -> false

int i = 0;
int r = co::atomic_cas(&i, 1, 2);   // i 不变, r -> 0
r = co::atomic_cas(&i, 0, 2);       // i -> 2, r -> 0

void* p = 0;
bool x = co::atomic_bool_cas(&p, 0, (void*)8);  // p -> 8, x -> true
```


## 加减

```cpp
template<typename T, typename V>
T atomic_add(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_sub(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T>
T atomic_inc(T* p, memorder_t mo = mo_seq_cst);

template<typename T>
T atomic_dec(T* p, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_fetch_add(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_fetch_sub(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T>
T atomic_fetch_inc(T* p, memorder_t mo = mo_seq_cst);

template<typename T>
T atomic_fetch_dec(T* p, memorder_t mo = mo_seq_cst);
```

- `inc` 加 1，`dec` 减 1。
- 带 fetch 版本返回旧值，不带 fetch 版本返回新值。
- 支持所有内存序。


## 位运算

```cpp
template<typename T, typename V>
T atomic_or(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_and(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_xor(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_fetch_or(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_fetch_and(T* p, V v, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
T atomic_fetch_xor(T* p, V v, memorder_t mo = mo_seq_cst);
```

- 带 fetch 版本返回旧值，不带 fetch 版本返回新值。
- 支持所有内存序。
