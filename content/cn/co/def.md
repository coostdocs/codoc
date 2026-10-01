---
weight: 1
title: "基本定义"
---


## 头文件

```cpp
#include "co/def.h"
```


## 整型别名

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

- 定义在**全局命名空间**。


## 整型极值常量

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

- 定义在 `co` 命名空间，均为 `constexpr`。

示例:

```cpp
#include "co/def.h"
#include "co/print.h"

int main() {
    co::println("max_uint32 = ", co::max_uint32);
    co::println("max_int32  = ", co::max_int32);
    co::println("min_int32  = ", co::min_int32);
    return 0;
}
```


## 缓存行大小

提供编译期常量 `co::cache_line_size`，常用于内存对齐、避免伪共享。各架构取值如下：

| 架构 | `co::cache_line_size` |
| --- | --- |
| S390X | 256 |
| PowerPC64 | 128 |
| ARM64 | 128 |
| 其它（x86、x64 等） | 64 |

{{< hint warning >}}
`co::cache_line_size` 可能大于实际缓存行大小，对齐或填充时会多占少量内存，通常影响较小。
{{< /hint >}}

示例:

```cpp
// 分配缓存行对齐的内存
co::alloc(n, co::cache_line_size);
```


## 宏

### 架构

```cpp
#if SIZE_MAX == UINT64_MAX
#define __arch64 1
#elif SIZE_MAX == UINT32_MAX
#define __arch32 1
#else
#error "platform not supported"
#endif
```

- 64 位平台定义 `__arch64` 宏，值为 1；
- 32 位平台定义 `__arch32` 宏，值为 1；

### 缓存行对齐

```cpp
#ifndef __cacheline_aligned
#define __cacheline_aligned alignas(co::cache_line_size)
#endif
```

- `__cacheline_aligned` 宏让变量或结构体按缓存行对齐。

示例：

```cpp
struct __cacheline_aligned Foo {
    int x;
};
```

### 线程局部存储

```cpp
#ifdef _MSC_VER
#ifndef __thread
#define __thread __declspec(thread)
#endif
#endif
```

- `__thread` 用于定义线程局部变量；
- gcc/clang 已内置 `__thread`，Windows 上(使用 MSVC)将其定义为 `__declspec(thread)`。

示例：

```cpp
__thread int g_v;
__thread void* g_p;
```

### `__unlikely`

- 提示编译器条件为假概率更高。
- 旧版本中名为 `unlikey`，为避免与 C++20 `[[unlikely]]` 冲突，改为 `__unlikely`。

示例：

```cpp
fs::file f("xx.log", 'r');
if (__unlikely(!f)) co::println("open file failed");
```
