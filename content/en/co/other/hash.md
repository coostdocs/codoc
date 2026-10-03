---
weight: 13
title: "Hash and Encoding"
---

## Header

```cpp
#include "co/base64.h"
#include "co/md5.h"
#include "co/murmur_hash.h"
#include "co/sha256.h"
```

The API is in the `co` namespace. All functions have overloads for `(const void*, size_t)`, `(const char*)`, `(const co::string&)`, and `(const std::string&)`.

## base64

```cpp
co::string co::base64_encode(const void* s, size_t n);
co::string co::base64_decode(const void* s, size_t n);
```

- base64 encoding / decoding.
- `decode` returns an empty string on error.

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

- Streaming interface, suitable for large files.

### md5digest

```cpp
void md5digest(const void* s, size_t n, char res[16]);
co::string md5digest(const void* s, size_t n);
```

- Outputs 16-byte binary.

### md5sum

```cpp
void md5sum(const void* s, size_t n, char res[32]);
co::string md5sum(const void* s, size_t n);
```

- Outputs a 32-byte hexadecimal string.

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

- Streaming interface.

### sha256digest

```cpp
void sha256digest(const void* s, size_t n, char res[32]);
co::string sha256digest(const void* s, size_t n);
```

- Outputs 32-byte binary.

### sha256sum

```cpp
void sha256sum(const void* s, size_t n, char res[64]);
co::string sha256sum(const void* s, size_t n);
```

- Outputs a 64-byte hexadecimal string.

## murmur_hash

```cpp
size_t co::murmur_hash(const void* s, size_t n);
```

- Non-cryptographic hash, returns `size_t`.
- Suitable for hashing and indexing; not for security scenarios.

## Example

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

    // streaming md5
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
