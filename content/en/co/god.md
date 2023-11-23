---
weight: 2
title: "God Oriented Programming"
---


include: [co/god.h](https://github.com/idealvin/coost/blob/master/include/co/god.h).


## god

The **god** module provides some features on templates. C++ code written with templates may be hard to understand. Some C++ programmers call it **god-oriented programming**.


### god::bless_no_bugs

```cpp
void bless_no_bugs();
```

- Pray to God to bless the code is bug-free. It is thread-safe, and can be called arbitrarily.

- example

```cpp
#include "co/god.h"
#include "co/cout.h"

int main(int argc, char** argv) {
     god::bless_no_bugs();
     co::print("hello world");
     return 0;
}
```



### god::cast

```cpp
template<typename To, typename From>
constexpr To cast(From&& f) {
     return (To) std::forward<From>(f);
}
```

- Universal conversion, convert `From` type to `To` type, `To` can be a reference.

- example

```cpp
int i = 65;
char c = god::cast<char>(i); // c -> 'A'
god::cast<char&>(i) = 'a';   // set the lower byte of i to 'a'
```



### ———————————
### god::align_down

```cpp
1. template<size_t A, typename X>
   constexpr X align_down(X x);

2. template<size_t A, typename X>
   constexpr X* align_down(X* x);

3. template<typename X, typename A>
   constexpr X align_down(X x, A a);

4. template<typename X, typename A>
   constexpr X* align_down(X* x, A a);
```

- Aligns an integer or pointer down.
- 1, align the integer `x` down by the compile-time constant `A`, which must be  power of 2.
- 2, Align pointer `x` down by compile-time constant `A`, `A` must be power of 2.
- 3, align the integer `x` down by the integer `a`, which must be power of 2.
- 4, Align pointer `x` down by integer `a`, `a` must be power of 2.

- example

```cpp
god::align_down<8>(30); // 24
god::align_down(30, 8); // 24
god::align_down<64>(30); // 0

void* p = (void*)40;
god::align_down<64>(p); // (void*)0
god::align_down<32>(p); // (void*)32
god::align_down(p, 32); // (void*)32
```



### god::align_up

```cpp
1. template<size_t A, typename X>
   constexpr X align_up(X x);

2. template<size_t A, typename X>
   constexpr X* align_up(X* x);

3. template<typename X, typename A>
   constexpr X align_up(X x, A a);

4. template<typename X, typename A>
   constexpr X* align_up(X* x, A a);
```

- Aligns an integer or pointer up.
- 1, align the integer `x` up by the compile-time constant `A`, `A` must be power of 2.
- 2, align the pointer `x` up by the compile-time constant `A`, `A` must be power of 2.
- 3, align integer `x` up by integer `a`, `a` must be power of 2.
- 4, align pointer `x` up by integer `a`, `a` must be power of 2.

- example

```cpp
god::align_up<8>(30); // 32
god::align_up(30, 8); // 32
god::align_up<64>(30); // 64

void* p = (void*)40;
god::align_up<64>(p); // (void*)64
god::align_up<32>(p); // (void*)64
god::align_up(p, 32); // (void*)64
```



### god::copy

```cpp
template<size_t N>
inline void copy(void* dst, const void* src);
```

- Copy `N` bytes from `src` to `dst`.

- example

```cpp
char s[8] = "1234567";
uint32 x;
god::copy<4>(&x, s + 1);
```



### god::eq

```cpp
template<typename T>
bool eq(const void* p, const void* q);
```

- Check whether `*(T*)p == *(T*)q` is true.

- example

```cpp
god::eq<uint32>("nicecode", "nicework"); // true
god::eq<uint64>("nicecode", "nicework"); // false
```



### god::log2

```cpp
template<typename T>
constexpr T log2(T x);
```

- Computes the base-2 logarithm of an integer x, rounding only the integer part of the result.

- example

```cpp
god::log2(32); // 5
god::log2(5); // 2
```



### god::nb

```cpp
template<size_t N, typename X>
constexpr X nb(X x);
```

- Calculate the number of blocks, `N` is the block size, `N` must be power of 2.

- example

```cpp
god::nb<8>(31); // 4
god::nb<16>(32); // 2
god::nb<32>(33); // 2
```



### ———————————
### god::fetch_add

```cpp
template<typename T, typename V>
inline T fetch_add(T* p, V v);
```

- Add `v` to the value of `*p`, return the original value.

- example

```cpp
int i = 8;
int x = god::fetch_add(&i, 1); // i -> 9, x -> 8
```



### god::fetch_sub

```cpp
template<typename T, typename V>
inline T fetch_sub(T* p, V v);
```

- Subtract `v` from the value of `*p`, return the original value.

- example

```cpp
int i = 8;
int x = god::fetch_sub(&i, 1); // i -> 7, x -> 8
```



### god::fetch_and

```cpp
template<typename T, typename V>
inline T fetch_and(T* p, V v);
```

- Perform the and operation on `*p`, return the original value.

- example

```cpp
uint32 u = 5;
uint32 x = god::fetch_and(&u, 1); // u -> 1, x -> 5
```



### god::fetch_or

```cpp
template<typename T, typename V>
inline T fetch_or(T* p, V v);
```

- Perform an or operation on `*p`, return the original value.

- example

```cpp
uint32 u = 2;
uint32 x = god::fetch_or(&u, 1); // u -> 3, x -> 2
```



### god::fetch_xor

```cpp
template<typename T, typename V>
inline T fetch_xor(T* p, V v);
```

- Perform the xor operation on `*p`, return the original value.

- example

```cpp
uint32 u = 2;
uint32 x = god::fetch_xor(&u, 2); // u -> 0, x -> 2
```



### god::swap

```cpp
template<typename T, typename V>
inline T swap(T* p, V v);
```

- Set the value of `*p` to `v`, return the original value.

- example

```cpp
uint32 u = 23;
uint32 x = god::swap(&u, 77); // u -> 77, x -> 23
```



### ———————————
### god::const_ref_t

```cpp
template<typename T>
using _const_t = typename std::add_const<T>::type;

template<typename T>
using const_ref_t = typename std::add_lvalue_reference<_const_t<rm_ref_t<T>>>::type;
```

- Added `const` and lvalue references.

- example

```cpp
god::const_ref_t<int>  ->  const int&
```



### god::if_t

```cpp
template<bool C, typename T=void>
using if_t = typename std::enable_if<C, T>::type;
```

- Equivalent to [std::enable_if_t](https://en.cppreference.com/w/cpp/types/enable_if).



### god::rm_arr_t

```cpp
template<typename T>
using rm_arr_t = typename std::remove_extent<T>::type;
```

- Remove the first array dimension of `T`.

- Example

```cpp
God::rm_arr_t<int[8]>     ->  int
god::rm_arr_t<int[6][8]>  ->  int[8]
```



### god::rm_cv_t

```cpp
template<typename T>
using rm_cv_t = typename std::remove_cv<T>::type;
```

- Remove `const` and `volatile` from `T`.



### god::rm_ref_t

```cpp
template<typename T>
using rm_ref_t = typename std::remove_reference<T>::type;
```

- Remove references in `T`.

- Example

```cpp
god::rm_ref_t<bool&>  ->  bool
god::rm_ref_t<int&&>  ->  int
```



### god::rm_cvref_t

```cpp
template<typename T>
using rm_cvref_t = rm_cv_t<rm_ref_t<T>>;
```

- Combination of [rm_cv_t](#godrm_cv_t) and [rm_ref_t](#godrm_ref_t).

- Example

```cpp
god::rm_cvref_t<const int&>  ->  int
```



### ———————————
### god::is_array

```cpp
template<typename T>
constexpr bool is_array();
```

- Determine whether `T` is an array.

- example

```cpp
god::is_array<char[8]>(); // true
god::is_array<char*>();   // false
```



### god::is_class

```cpp
template<typename T>
constexpr bool is_class();
```

- Determine whether `T` is a class.

- example

```cpp
god::is_class<std::string>(); // true
```



### god::is_ref

```cpp
template<typename T>
constexpr bool is_ref();
```

- Determine whether `T` is a reference.

```cpp
god::is_ref<const int&>(); // true
god::is_ref<int&&>();      // true
```



### god::is_same

```cpp
template<typename T, typename U, typename ... X>
constexpr bool is_same();
```

- Determine whether `T` is one of `U` or `X...`.

- example

```cpp
god::is_same<int, int>();        // true
god::is_same<int, bool, int>();  // true
god::is_same<int, bool, char>(); // false
```



### god::is_scalar

```cpp
template<typename T>
constexpr bool is_scalar();
```

- Determine whether `T` is a scalar type.