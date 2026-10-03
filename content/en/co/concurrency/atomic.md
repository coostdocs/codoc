---
weight: 1
title: "Atomic Operations"
---

## Header

```cpp
#include "co/atomic.h"
```

The API is in the `co` namespace.

## Memory Order

```cpp
using memorder_t = std::memory_order;
constexpr memorder_t mo_relaxed = std::memory_order_relaxed;
constexpr memorder_t mo_consume = std::memory_order_consume;
constexpr memorder_t mo_acquire = std::memory_order_acquire;
constexpr memorder_t mo_release = std::memory_order_release;
constexpr memorder_t mo_acq_rel = std::memory_order_acq_rel;
constexpr memorder_t mo_seq_cst = std::memory_order_seq_cst;
```

## Load and Store

```cpp
template<typename T>
T atomic_load(const T* p, memorder_t mo = mo_seq_cst);

template<typename T, typename V>
void atomic_store(T* p, V v, memorder_t mo = mo_seq_cst);
```

- `atomic_load` supports: `mo_relaxed`, `mo_consume`, `mo_acquire`, `mo_seq_cst`.
- `atomic_store` supports: `mo_relaxed`, `mo_release`, `mo_seq_cst`.

Example:

```cpp
int i = 0;
co::atomic_store(&i, 3);      // i -> 3
int x = co::atomic_load(&i);  // x -> 3
```

## Exchange

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

- `atomic_swap` returns the old value and supports all memory orders.
- `atomic_cas` is equivalent to `atomic_compare_swap`; it performs the swap only when `*p == o`, and returns the old value.
- `atomic_bool_cas` is similar to `atomic_cas`; it returns true on success and false on failure.
- In CAS operations, `smo` is the memory order on success and supports all memory orders; `fmo` is the memory order on failure, cannot be `mo_release` or `mo_acq_rel`, and cannot be stronger than `smo`.

Example:

```cpp
bool b = false;
bool o = co::atomic_swap(&b, true); // b -> true, o -> false

int i = 0;
int r = co::atomic_cas(&i, 1, 2);   // i unchanged, r -> 0
r = co::atomic_cas(&i, 0, 2);       // i -> 2, r -> 0

void* p = 0;
bool x = co::atomic_bool_cas(&p, 0, (void*)8);  // p -> 8, x -> true
```

## Add and Subtract

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

- `inc` adds 1, `dec` subtracts 1.
- The fetch versions return the old value; the non-fetch versions return the new value.
- Supports all memory orders.

## Bitwise Operations

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

- The fetch versions return the old value; the non-fetch versions return the new value.
- Supports all memory orders.
