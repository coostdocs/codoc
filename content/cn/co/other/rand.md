---
weight: 7
title: "随机值"
---


## 头文件

```cpp
#include "co/rand.h"
```

API 在 `co` 命名空间，无需 `flag::parse`。


## 随机数

```cpp
// 线程安全，返回 0 < x < 2^31-1
uint32 co::rand();

// seed 必须在 (0, 2^31-1)，调用后更新为返回值
uint32 co::rand(uint32& seed);

// 线程安全，返回 64 位随机数
uint64 co::rand64();

// splitmix64，seed 可为 0，调用后更新
uint64 co::rand64(uint64& seed);
```

- `rand()` / `rand64()` 线程安全，内部使用 thread_local 状态。
- `rand(seed)` 的 seed 不能为 0，否则后续返回值恒为 0；也不能为 `2^31-1`。
- `rand64(seed)` 对 seed 初值无要求。
- 带 seed 的版本线程安全取决于调用方对 seed 的使用。


## 随机字符串

```cpp
// 写随机字符到 buf，末尾不添加 '\0'
void co::randchars(void* buf, size_t bufsize);

// 返回长度 n 的随机字符串，默认 15，线程安全
co::string co::randstr(uint32 n=15);

// 使用指定字符集，支持 "0-9"、"a-f" 等范围
co::string co::randstr(const char* charset, uint32 n);
```

- `randstr()` 线程安全，内部使用 thread_local 状态。
- `randchars` 默认字符集：`a-z`、`A-Z`、`0-9`、`_`、`-`，共 64 个字符。
- `randstr(charset, n)` 支持范围缩写，如 `"0-9"`、`"a-f"`、`"0-9A-Za-z"`。
- `randchars` 不写结尾 `'\0'`。


## 示例

```cpp
#include "co/rand.h"
#include "co/print.h"

int main() {
    // 随机数
    co::println("rand()   = ", co::rand());
    co::println("rand64() = ", co::rand64());

    uint32 seed = 12345;
    co::println("rand(seed) = ", co::rand(seed));
    co::println("seed = ", seed);

    uint64 seed64 = 0;
    co::println("rand64(seed) = ", co::rand64(seed64));
    co::println("seed64 = ", seed64);

    // 随机字符串
    co::println("randstr()        = ", co::randstr());
    co::println("randstr(8)       = ", co::randstr(8));
    co::println("randstr(0-9, 8)  = ", co::randstr("0-9", 8));
    co::println("randstr(0-9a-f)  = ", co::randstr("0-9a-f", 8));

    char buf[9];
    co::randchars(buf, 8);
    buf[8] = '\0';
    co::println("randchars = ", buf);
    return 0;
}
```
