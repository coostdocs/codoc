---
weight: 2
title: "原子操作"
---

include: [co/atomic.h](https://github.com/idealvin/co/tree/master/include/co/atomic.h).


## 算术运算


### atomic_inc

```cpp
template<typename T>
inline T atomic_inc(T* p);
```

- 原子自增，T 是长度为 1, 2, 4, 8 字节的任意整数类型，参数 p 是 T 类型的指针。
- 此函数对 p 指向的整数进行自增操作，并返回自增后的结果。

- 示例

```cpp
int i = 0;
uint64 u = 0;
int r = atomic_inc(&i);     // i -> 1, r = 1
uint64 x = atomic_inc(&u);  // u -> 1, x = 1
```



### atomic_fetch_inc

```cpp
template<typename T>
inline T atomic_fetch_inc(T* p);
```

- 与 [atomic_inc](#atomic_inc) 一样，但返回自增前的值。

- 示例

```cpp
int i = 0;
uint64 u = 0;
int r = atomic_fetch_inc(&i);     // i -> 1, r = 0
uint64 x = atomic_fetch_inc(&u);  // u -> 1, x = 0
```



### atomic_dec

```cpp
template<typename T>
inline T atomic_dec(T* p);
```

- 原子自减，T 是长度为 1, 2, 4, 8 字节的任意整数类型，参数 p 是 T 类型的指针。
- 此函数对 p 指向的整数进行自减操作，并返回自减后的结果。

- 示例

```cpp
int i = 1;
uint64 u = 1;
int r = atomic_dec(&i);     // i -> 0, r = 0
uint64 x = atomic_dec(&u);  // u -> 0, x = 0
```



### atomic_fetch_dec

```cpp
template<typename T>
inline T atomic_fetch_dec(T* p);
```

- 与 [atomic_dec](#atomic_dec) 一样，但返回自减前的值。

- 示例

```cpp
int i = 1;
uint64 u = 1;
int r = atomic_fetch_dec(&i);     // i -> 0, r = 1
uint64 x = atomic_fetch_dec(&u);  // u -> 0, x = 1
```



### atomic_add

```cpp
template<typename T, typename V>
inline T atomic_add(T* p, V v);
```

- 原子加法，T 是长度为 1, 2, 4, 8 字节的任意整数类型，V 是任意整数类型，参数 p 是 T 类型的指针。
- 此函数对 p 指向的整数加上值 v，并返回加 v 后的结果。

- 示例

```cpp
int i = 0;
uint64 u = 0;
int r = atomic_add(&i, 1);     // i -> 1, r = 1
uint64 x = atomic_add(&u, 1);  // u -> 1, x = 1
```



### atomic_fetch_add

```cpp
template<typename T, typename V>
inline T atomic_fetch_add(T* p, V v);
```

- 与 [atomic_add](#atomic_add) 一样，但返回加 v 前的值。

- 示例

```cpp
int i = 0;
uint64 u = 0;
int r = atomic_fetch_add(&i, 1);     // i -> 1, r = 0
uint64 x = atomic_fetch_add(&u, 1);  // u -> 1, x = 0
```



### atomic_sub

```cpp
template<typename T, typename V>
inline T atomic_sub(T* p, V v);
```

- 原子减法，T 是长度为 1, 2, 4, 8 字节的任意整数类型，V 是任意整数类型，参数 p 是 T 类型的指针。
- 此函数对 p 指向的整数减去值 v，并返回减 v 后的结果。

- 示例

```cpp
int i = 1;
uint64 u = 1;
int r = atomic_sub(&i, 1);     // i -> 0, r = 0
uint64 x = atomic_sub(&u, 1);  // u -> 0, x = 0
```



### atomic_fetch_sub

```cpp
template<typename T, typename V>
inline T atomic_fetch_sub(T* p, V v);
```

- 与 [atomic_sub](#atomic_sub) 一样，但返回减 v 前的值。

- 示例

```cpp
int i = 1;
uint64 u = 1;
int r = atomic_fetch_sub(&i, 1);     // i -> 0, r = 1
uint64 x = atomic_fetch_sub(&u, 1);  // u -> 0, x = 1
```




## 位运算


### atomic_or

```cpp
template<typename T, typename V>
inline T atomic_or(T* p, V v);
```

- 原子位或，T 是长度为 1, 2, 4, 8 字节的任意整数类型，V 是任意整数类型，参数 p 是 T 类型的指针。
- 此函数对 p 指向的整数与 v 进行位或操作，并返回操作后的结果。

- 示例

```cpp
int i = 5;
uint64 u = 5;
int r = atomic_or(&i, 3);     // i |= 3, i -> 7, r = 7
uint64 x = atomic_or(&u, 3);  // u |= 3, u -> 7, x = 7
```



### atomic_fetch_or

```cpp
template<typename T, typename V>
inline T atomic_fetch_or(T* p, V v);
```

- 与 [atomic_or](#atomic_or) 一样，但返回位或操作之前的值。

- 示例

```cpp
int i = 5;
uint64 u = 5;
int r = atomic_fetch_or(&i, 3);     // i |= 3, i -> 7, r = 5
uint64 x = atomic_fetch_or(&u, 3);  // u |= 3, u -> 7, x = 5
```



### atomic_and

```cpp
template<typename T, typename V>
inline T atomic_and(T* p, V v);
```

- 原子位与，T 是长度为 1, 2, 4, 8 字节的任意整数类型，V 是任意整数类型，参数 p 是 T 类型的指针。
- 此函数对 p 指向的整数与 v 进行位与操作，并返回操作后的结果。

- 示例

```cpp
int i = 5;
uint64 u = 5;
int r = atomic_and(&i, 3);     // i &= 3, i -> 1, r = 1
uint64 x = atomic_and(&u, 3);  // u &= 3, u -> 1, x = 1
```



### atomic_fetch_and

```cpp
template<typename T, typename V>
inline T atomic_fetch_and(T* p, V v);
```

- 与 [atomic_and](#atomic_and) 一样，但返回位与操作之前的值。

- 示例

```cpp
int i = 5;
uint64 u = 5;
int r = atomic_fetch_and(&i, 3);     // i &= 3, i -> 1, r = 5
uint64 x = atomic_fetch_and(&u, 3);  // u &= 3, u -> 1, x = 5
```



### atomic_xor

```cpp
template<typename T, typename V>
inline T atomic_xor(T* p, V v);
```

- 原子按位异或，T 是长度为 1, 2, 4, 8 字节的任意整数类型，V 是任意整数类型，参数 p 是 T 类型的指针。
- 此函数对 p 指向的整数与 v 进行按位异或操作，并返回操作后的结果。

- 示例

```cpp
int i = 5;
uint64 u = 5;
int r = atomic_xor(&i, 3);     // i ^= 3, i -> 6, r = 6
uint64 x = atomic_xor(&u, 3);  // u ^= 3, u -> 6, x = 6
```



### atomic_fetch_xor

```cpp
template<typename T, typename V>
inline T atomic_fetch_xor(T* p, V v);
```

- 与 [atomic_xor](#atomic_xor) 一样，但返回按位异或操作之前的值。

- 示例

```cpp
int i = 5;
uint64 u = 5;
int r = atomic_fetch_xor(&i, 3);     // i ^= 3, i -> 6, r = 5
uint64 x = atomic_fetch_xor(&u, 3);  // u ^= 3, u -> 6, x = 5
```




## 交换


### atomic_swap

```cpp
template<typename T, typename V>
inline T atomic_swap(T* p, V v);
```

- 原子交换操作，T 是长度为 1, 2, 4, 8 字节的任意内置数据类型(包括指针类型)，V 是可以转换为 T 类型的任意类型。
- 此函数对 p 指向的值与 v 进行交换操作，并返回交换操作前的值。

- 示例

```cpp
bool b = false;
int i = 0;
void* p = 0;
bool x = atomic_swap(&b, true);       // b -> true, x = false
int r = atomic_swap(&i, 1);           // i -> 1, r = 0
void* q = atomic_swap(&p, (void*)8);  // p -> 8, q = 0
```



### atomic_compare_swap

```cpp
template<typename T, typename O, typename V>
inline T atomic_compare_swap(T* p, O o, V v);
```

- 原子交换操作，T 是长度为 1, 2, 4, 8 字节的任意内置数据类型(包括指针类型)，O 与 V 是可以转换为 T 类型的任意类型。
- 此函数仅在 p 指向的值与 o 相等时，才与 v 进行交换操作。
- 此函数返回交换操作前的值，用户可以根据返回值是否与 o 相等来判断是否进行了交换操作。

- 示例

```cpp
bool b = false;
int i = 0;
void* p = 0;
bool x = atomic_compare_swap(&b, false, true);   // b -> true, x = false
int r = atomic_compare_swap(&i, 1, 2);           // 不会交换, i 保持不变, r = 0
void* q = atomic_compare_swap(&p, 0, (void*)8);  // p -> 8, q = 0
```




## get & set


### atomic_get

```cpp
template<typename T>
inline T atomic_get(T* p);
```

- 此函数获取 p 指向的变量的值，T 是长度为 1, 2, 4, 8 字节的任意内置数据类型(包括指针类型)。

- 示例

```cpp
int i = 7;
int r = atomic_get(&i);  // r = 7
```



### atomic_set

```cpp
template<typename T, typename V>
inline void atomic_set(T* p, V v);
```

- 此函数将 p 指向的值设为 v，T 是长度为 1, 2, 4, 8 字节的任意内置数据类型(包括指针类型)，V 是可以转换为 T 类型的任意类型。

- 示例

```cpp
int i = 7;
atomic_set(&i, 3);  // i -> 3
```



### atomic_reset

```cpp
template<typename T>
inline void atomic_reset(T* p);
```

- 此函数将 p 指向的值设为 0，T 是长度为 1, 2, 4, 8 字节的任意内置数据类型(包括指针类型)。

- 示例

```cpp
int i = 7;
void* p = &i;
atomic_reset(&i); // i -> 0
atomic_reset(&p); // p -> 0
```
