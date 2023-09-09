---
weight: 2
title: "面向玄学"
---


include: [co/god.h](https://github.com/idealvin/coost/blob/master/include/co/god.h).


## god

**god** 模块提供了一些与模板相关的功能，模板用到深处有点玄，一些 C++ 程序员将之称为**面向玄学编程**。


### god::bless_no_bugs

```cpp
void bless_no_bugs();
```

- 祈求老天保佑代码无 bug，线程安全，可任意调用。

- 示例

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

- 万能转换，将 `From` 类型转换为 `To` 类型，`To` 可以是引用。

- 示例

```cpp
int i = 65;
char c = god::cast<char>(i);  // c -> 'A'
god::cast<char&>(i) = 'a';    // 将 i 的低字节设置为 'a'
```



### ---------------
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

- 将整数或指针向下对齐。
- 1, 将整数 `x` 按编译期常量 `A` 向下对齐，`A` 必须是 2 的幂。
- 2, 将指针 `x` 按编译期常量 `A` 向下对齐，`A` 必须是 2 的幂。
- 3, 将整数 `x` 按整数 `a` 向下对齐，`a` 必须是 2 的幂。
- 4, 将指针 `x` 按整数 `a` 向下对齐，`a` 必须是 2 的幂。

- 示例

```cpp
god::align_down<8>(30);  // 24
god::align_down(30, 8);  // 24
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

- 将整数或指针向上对齐。
- 1, 将整数 `x` 按编译期常量 `A` 向上对齐，`A` 必须是 2 的幂。
- 2, 将指针 `x` 按编译期常量 `A` 向上对齐，`A` 必须是 2 的幂。
- 3, 将整数 `x` 按整数 `a` 向上对齐，`a` 必须是 2 的幂。
- 4, 将指针 `x` 按整数 `a` 向上对齐，`a` 必须是 2 的幂。

- 示例

```cpp
god::align_up<8>(30);  // 32
god::align_up(30, 8);  // 32
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

- 从 `src` 拷贝 `N` 字节到 `dst` 中。

- 示例

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

- 判断 `*(T*)p == *(T*)q` 是否成立。

- 示例

```cpp
god::eq<uint32>("nicecode", "nicework"); // true
god::eq<uint64>("nicecode", "nicework"); // false
```



### god::log2

```cpp
template<typename T>
constexpr T log2(T x);
```

- 计算整数 x 的以 2 为底的对数，结果仅取整数部分。

- 示例

```cpp
god::log2(32); // 5
god::log2(5);  // 2
```



### god::nb

```cpp
template<size_t N, typename X>
constexpr X nb(X x);
```

- 计算 block 数，`N` 是 block 大小，`N` 必须是 2 的幂。

- 示例

```cpp
god::nb<8>(31);   // 4
god::nb<16>(32);  // 2
god::nb<32>(33);  // 2
```



### ---------------
### god::fetch_add

```cpp
template<typename T, typename V>
inline T fetch_add(T* p, V v);
```

- 将 `*p` 的值加上 `v`，返回原来的值。

- 示例

```cpp
int i = 8;
int x = god::fetch_add(&i, 1); // i -> 9, x -> 8
```



### god::fetch_sub

```cpp
template<typename T, typename V>
inline T fetch_sub(T* p, V v);
```

- 将 `*p` 的值减去 `v`，返回原来的值。

- 示例

```cpp
int i = 8;
int x = god::fetch_sub(&i, 1); // i -> 7, x -> 8
```



### god::fetch_and

```cpp
template<typename T, typename V>
inline T fetch_and(T* p, V v);
```

- 对 `*p` 执行 and 操作，返回原来的值。

- 示例

```cpp
uint32 u = 5;
uint32 x = god::fetch_and(&u, 1); // u -> 1, x -> 5
```



### god::fetch_or

```cpp
template<typename T, typename V>
inline T fetch_or(T* p, V v);
```

- 对 `*p` 执行 or 操作，返回原来的值。

- 示例

```cpp
uint32 u = 2;
uint32 x = god::fetch_or(&u, 1); // u -> 3, x -> 2
```



### god::fetch_xor

```cpp
template<typename T, typename V>
inline T fetch_xor(T* p, V v);
```

- 对 `*p` 执行 xor 操作，返回原来的值。

- 示例

```cpp
uint32 u = 2;
uint32 x = god::fetch_xor(&u, 2); // u -> 0, x -> 2
```



### god::swap

```cpp
template<typename T, typename V>
inline T swap(T* p, V v);
```

- 将 `*p` 的值设置为 `v`，返回原来的值。

- 示例

```cpp
uint32 u = 23;
uint32 x = god::swap(&u, 77); // u -> 77, x -> 23
```



### ---------------
### god::const_ref_t

```cpp
template<typename T>
using _const_t = typename std::add_const<T>::type;

template<typename T>
using const_ref_t = typename std::add_lvalue_reference<_const_t<rm_ref_t<T>>>::type;
```

- 添加 `const` 与左值引用。

- 示例

```cpp
god::const_ref_t<int>  ->  const int&
```



### god::if_t

```cpp
template<bool C, typename T=void>
using if_t = typename std::enable_if<C, T>::type;
```

- 等价于 [std::enable_if_t](https://en.cppreference.com/w/cpp/types/enable_if)。



### god::rm_arr_t

```cpp
template<typename T>
using rm_arr_t = typename std::remove_extent<T>::type;
```

- 移除 `T` 的第一个数组维度。

- 示例

```cpp
god::rm_arr_t<int[8]>     ->  int
god::rm_arr_t<int[6][8]>  ->  int[8]
```



### god::rm_cv_t

```cpp
template<typename T>
using rm_cv_t = typename std::remove_cv<T>::type;
```

- 移除 `T` 中的 `const` 与 `volatile`。



### god::rm_ref_t

```cpp
template<typename T>
using rm_ref_t = typename std::remove_reference<T>::type;
```

- 移除 `T` 中的引用。

- 示例

```cpp
god::rm_ref_t<bool&>  ->  bool
god::rm_ref_t<int&&>  ->  int
```



### god::rm_cvref_t

```cpp
template<typename T>
using rm_cvref_t = rm_cv_t<rm_ref_t<T>>;
```

- [rm_cv_t](#godrm_cv_t) 与 [rm_ref_t](#godrm_ref_t) 的组合。

- 示例

```cpp
god::rm_cvref_t<const int&>  ->  int
```



### ---------------
### god::is_array

```cpp
template<typename T>
constexpr bool is_array();
```

- 判断 `T` 是否为数组类型。

- 示例

```cpp
god::is_array<char[8]>();  // true
god::is_array<char*>();    // false
```



### god::is_class

```cpp
template<typename T>
constexpr bool is_class();
```

- 判断 `T` 是否为 class。

- 示例

```cpp
god::is_class<std::string>(); // true
```



### god::is_ref

```cpp
template<typename T>
constexpr bool is_ref();
```

- 判断 `T` 是否为引用。

```cpp
god::is_ref<const int&>();  // true
god::is_ref<int&&>();       // true
```



### god::is_same

```cpp
template<typename T, typename U, typename ...X>
constexpr bool is_same();
```

- 判断类型 `T` 是否为 `U` 或 `X...` 中的一个。

- 示例

```cpp
god::is_same<int, int>();         // true
god::is_same<int, bool, int>();   // true
god::is_same<int, bool, char>();  // false
```



### god::is_scalar

```cpp
template<typename T>
constexpr bool is_scalar();
```

- 判断 `T` 是否为标量类型。
