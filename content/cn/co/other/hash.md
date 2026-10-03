---
weight: 13
title: 哈希与编码
---


## 头文件

```cpp
#include "co/base64.h"
#include "co/md5.h"
#include "co/murmur_hash.h"
#include "co/sha256.h"
```

API 在 `co` 命名空间。所有函数都有 `(const void*, size_t)`、`(const char*)`、`(const co::string&)`、`(const std::string&)` 重载。


## base64

```cpp
co::string co::base64_encode(const void* s, size_t n);
co::string co::base64_decode(const void* s, size_t n);
```

- base64 编码 / 解码。
- `decode` 出错返回空字符串。


## md5

```cpp
typedef struct {
    uint32 lo, hi;
    uint32 a, b, c, d;
    uint8  buffer[64];
    uint32 block[16];
} md5_ctx_t;

void md5_init(md5_ctx_t* ctx);
void md5_update(md5_ctx_t* ctx, const void* s, size_t n);
void md5_final(md5_ctx_t* ctx, uint8 res[16]);
```

- 流式接口，适合大文件。

### md5digest

```cpp
void md5digest(const void* s, size_t n, char res[16]);
co::string md5digest(const void* s, size_t n);
```

- 输出 16 字节二进制。

### md5sum

```cpp
void md5sum(const void* s, size_t n, char res[32]);
co::string md5sum(const void* s, size_t n);
```

- 输出 32 字节十六进制字符串。


## sha256

```cpp
typedef struct {
    uint32 state[8];
    uint64 count;
    uint8  buffer[64];
} sha256_ctx_t;

void sha256_init(sha256_ctx_t* ctx);
void sha256_update(sha256_ctx_t* ctx, const void* s, size_t n);
void sha256_final(sha256_ctx_t* ctx, uint8 res[32]);
```

- 流式接口。

### sha256digest

```cpp
void sha256digest(const void* s, size_t n, char res[32]);
co::string sha256digest(const void* s, size_t n);
```

- 输出 32 字节二进制。

### sha256sum

```cpp
void sha256sum(const void* s, size_t n, char res[64]);
co::string sha256sum(const void* s, size_t n);
```

- 输出 64 字节十六进制字符串。


## murmur_hash

```cpp
size_t co::murmur_hash(const void* s, size_t n);
```

- 非加密哈希，返回 `size_t`。
- 适合散列、索引，不用于安全场景。

## 示例

```cpp
#include "co/base64.h"
#include "co/md5.h"
#include "co/sha256.h"
#include "co/murmur_hash.h"
#include "co/print.h"

int main() {
    const char* s = "hello";

    co::println("base64 = ", co::base64_encode(s));
    co::println("decode = ", co::base64_decode("aGVsbG8="));

    co::println("md5sum  = ", co::md5sum(s));
    co::println("sha256sum = ", co::sha256sum(s));

    co::println("murmur = ", co::murmur_hash(s, strlen(s)));

    // 流式 md5
    md5_ctx_t ctx;
    co::md5_init(&ctx);
    co::md5_update(&ctx, "he", 2);
    co::md5_update(&ctx, "llo", 3);
    char res[16];
    co::md5_final(&ctx, (uint8*)res);
    co::println("md5digest = ", co::string(res, 16));

    return 0;
}
```
