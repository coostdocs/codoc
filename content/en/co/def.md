---
weight: 1
title: "Basic Definitions"
---

## Header

```cpp
#include "co/def.h"
```

## Integer Type Aliases

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

- Defined in the **global namespace**.

## Integer Extreme Value Constants

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

- Defined in the `co` namespace; all are `constexpr`.

Example:

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

## Cache Line Size

Provides the compile-time constant `co::cache_line_size`, commonly used for memory alignment and avoiding false sharing. Values for each architecture are as follows:

| Architecture | `co::cache_line_size` |
| --- | --- |
| S390X | 256 |
| PowerPC64 | 128 |
| ARM64 | 128 |
| Others (x86, x64, etc.) | 64 |

{{< hint warning >}}
`co::cache_line_size` may be larger than the actual cache line size. When aligning or padding, it will occupy a small amount of extra memory, which usually has little impact.
{{< /hint >}}

Example:

```cpp
// Allocate cache-line-aligned memory
co::alloc(n, co::cache_line_size);
```

## Macros

### Architecture

```cpp
#if SIZE_MAX == UINT64_MAX
#define __arch64 1
#elif SIZE_MAX == UINT32_MAX
#define __arch32 1
#else
#error "platform not supported"
#endif
```

- On 64-bit platforms, the `__arch64` macro is defined with a value of 1;
- On 32-bit platforms, the `__arch32` macro is defined with a value of 1;

### Cache Line Alignment

```cpp
#ifndef __cacheline_aligned
#define __cacheline_aligned alignas(co::cache_line_size)
#endif
```

- The `__cacheline_aligned` macro aligns a variable or struct to a cache line.

Example:

```cpp
struct __cacheline_aligned Foo {
    int x;
};
```

### Thread-Local Storage

```cpp
#ifdef _MSC_VER
#ifndef __thread
#define __thread __declspec(thread)
#endif
#endif
```

- `__thread` is used to define thread-local variables;
- gcc/clang already have built-in `__thread`; on Windows (using MSVC), it is defined as `__declspec(thread)`.

Example:

```cpp
__thread int g_v;
__thread void* g_p;
```

### `__unlikely`

- Hints to the compiler that a condition is more likely to be false.
- In older versions it was named `unlikey`; to avoid conflict with C++20 `[[unlikely]]`, it was changed to `__unlikely`.

Example:

```cpp
fs::file f("xx.log", 'r');
if (__unlikely(!f)) co::println("open file failed");
```
