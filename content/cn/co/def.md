
weight: 1
title: "基本定义"



## 头文件

```cpp
#include "co/def.h"
```

coost 很多组件都会包含这个头文件。


## 整型别名

以下类型别名定义在**全局命名空间**，不在 `co` 命名空间：

```cpp
typedef int8_t  int8;
typedef int16_t int16;
typedef int32_t int32;
typedef int64_t int64;
typedef uint8_t  uint8;
typedef uint16_t uint16;
typedef uint32_t uint32;
typedef uint64_t uint64;
```

说明：

- 这些别名在 coost 中广泛使用，例如 `co::rand()` 返回 `uint32`，`co::now.ns()` 返回 `int64`。
- 因为定义在全局命名空间，所以直接写 `int32`、`uint64` 即可，不需要 `co::` 前缀。



## 整型极值常量

定义在 `co` 命名空间，均为 `constexpr`：

```cpp
namespace co {

constexpr uint8  max_uint8  = (uint8)  ~((uint8) 0);
constexpr uint16 max_uint16 = (uint16) ~((uint16)0);
constexpr uint32 max_uint32 = (uint32) ~((uint32)0);
constexpr uint64 max_uint64 = (uint64) ~((uint64)0);
constexpr int8  max_int8  = (int8)  (max_uint8  >> 1);
constexpr int16 max_int16 = (int16) (max_uint16 >> 1);
constexpr int32 max_int32 = (int32) (max_uint32 >> 1);
constexpr int64 max_int64 = (int64) (max_uint64 >> 1);
constexpr int8  min_int8  = (int8)  ~max_int8;
constexpr int16 min_int16 = (int16) ~max_int16;
constexpr int32 min_int32 = (int32) ~max_int32;
constexpr int64 min_int64 = (int64) ~max_int64;

} // co
```



## 缓存行大小

```cpp
namespace co {

#if defined(__s390x__)
constexpr int cache_line_size = 256;
#elif defined(__powerpc64__) || defined(_M_PPC64)
constexpr int cache_line_size = 128;
#elif defined(__aarch64__) || defined(_M_ARM64)
constexpr int cache_line_size = 128;
#else
constexpr int cache_line_size = 64;
#endif

} // co
```

各架构对应值：

| 架构 | 缓存行大小 |
| --- | --- |
| `__s390x__` | 256 |
| `__powerpc64__` / `_M_PPC64` | 128 |
| `__aarch64__` / `_M_ARM64` | 128 |
| 其它（x86、x64 等） | 64 |




## 宏

### `__arch64 & __arch32`

```cpp
#if SIZE_MAX == UINT64_MAX
#define __arch64 1
#elif SIZE_MAX == UINT32_MAX
#define __arch32 1
#else
#error "platform not supported"
#endif
```

说明：

- 64 位平台定义 `__arch64`；
- 32 位平台定义 `__arch32`；
- 其它情况编译期报错。




### `__cacheline_aligned`

```cpp
#ifndef __cacheline_aligned
#define __cacheline_aligned alignas(co::cache_line_size)
#endif
```

用于让变量或结构体按缓存行对齐。

示例：

```cpp
struct __cacheline_aligned Foo {
    int x;
};
```

### `__thread`

说明：

- 用于定义线程局部对象。

示例：

```cpp
__thread int g_v;
__thread void* g_p;
```

### `__unlikely`

```cpp
#ifndef __unlikely
#if (defined(__GNUC__) && __GNUC__ >= 3) || defined(__clang__)
#define __unlikely(x) (__builtin_expect(!!(x), 0))
#else
#define __unlikely(x) (x)
#endif
#endif
```

说明：

- 提示编译器 `x` 为假的概率更高。
- 旧版本中名为 `unlikey`，为避免与 C++20 中的 `[[unlikely]]` 属性冲突，改为 `__unlikely`。



## 示例

```cpp
#include "co/def.h"
#include "co/print.h"

int main() {
    co::println("max_uint32 = ", co::max_uint32);
    co::println("max_int32  = ", co::max_int32);
    co::println("min_int32  = ", co::min_int32);
    co::println("cache_line_size = ", co::cache_line_size);

#if defined(__arch64)
    co::println("arch: 64-bit");
#elif defined(__arch32)
    co::println("arch: 32-bit");
#endif

    return 0;
}
```
