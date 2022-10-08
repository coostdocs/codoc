---
weight: 20
title: "Hash"
---


include: [co/hash.h](https://github.com/idealvin/coost/blob/master/include/co/hash.h).


## Hash

### hash32

```cpp
uint32 hash32(const void* s, size_t n);
uint32 hash32(const char* s);
uint32 hash32(const fastring& s)
uint32 hash32(const std::string& s);
```

- 此函数返回 32 位的 murmur hash 值。
- s 为指针时，一般要求是 `sizeof(void*)` 字节对齐的。



### hash64

```cpp
uint64 hash64(const void* s, size_t n);
uint64 hash64(const char* s);
uint64 hash64(const fastring& s);
uint64 hash64(const std::string& s);
```

- 此函数返回 64 位的 murmur hash 值。
- s 为指针时，一般要求是 8 字节对齐的。



### murmur_hash

```cpp
size_t murmur_hash(const void* s, size_t n);
```

- 此函数返回 `size_t` 类型的 hash 值，这个值在 64 位平台是 64 位的，在 32 位平台是 32 位的。
- 参数 s 一般要求是 `sizeof(void*)` 字节对齐的。




## md5

### md5digest

```cpp
void md5digest(const void* s, size_t n, char res[16]);
fastring md5digest(const void* s, size_t n);
fastring md5digest(const char* s);
fastring md5digest(const fastring& s);
fastring md5digest(const std::string& s);
```

- 计算字符串的 md5 值，结果为 16 字节的二进制字符串。



### md5sum

```cpp
void md5sum(const void* s, size_t n, char res[32]);
fastring md5sum(const void* s, size_t n);
fastring md5sum(const char* s);
fastring md5sum(const fastring& s);
fastring md5sum(const std::string& s);
```

- 计算字符串的 md5 值，结果为 32 字节、仅含十六进制字符(0-9,a-f)的字符串。



### Lower level APIs

```cpp
void md5_init(md5_ctx_t* ctx);
void md5_update(md5_ctx_t* ctx, const void* s, size_t n);
void md5_final(md5_ctx_t* ctx, uint8 res[16]);
```

- 上述 3 个 API 可用于增量计算 md5。


- 示例

```cpp
char buf[4096];
uint8 res[16];
md5_ctx_t ctx;
md5_init(&ctx);

while (true) {
    int r = read(fd, buf, 4096);
    if (r > 0) {
        md5_update(&ctx, buf, r);
    } else {
        break;
    }
}

md5_final(&ctx, res);
```




## sha256

### sha256digest

```cpp
void sha256digest(const void* s, size_t n, char res[32]);
fastring sha256digest(const void* s, size_t n);
fastring sha256digest(const char* s);
fastring sha256digest(const fastring& s);
fastring sha256digest(const std::string& s);
```

- 计算字符串的 sha256 值，结果为 32 字节的二进制字符串。



### sha256sum

```cpp
void sha256sum(const void* s, size_t n, char res[64]);
fastring sha256sum(const void* s, size_t n);
fastring sha256sum(const char* s);
fastring sha256sum(const fastring& s);
fastring sha256sum(const std::string& s);
```

- 计算字符串的 sha256 值，结果为 64 字节、仅含十六进制字符(0-9,a-f)的字符串。



### Lower level APIs

```cpp
void sha256_init(sha256_ctx_t* ctx);
void sha256_update(sha256_ctx_t* ctx, const void* s, size_t n);
void sha256_final(sha256_ctx_t* ctx, uint8 res[32]);
```

- 上述 3 个 API 与 md5 类似，可用于增量计算 sha256。




## base64

### base64_encode

```cpp
fastring base64_encode(const void* s, size_t n);
fastring base64_encode(const char* s);
fastring base64_encode(const fastring& s);
fastring base64_encode(const std::string& s);
```

- `base64` 编码，实现中不添加 `\r\n`，实际应用中，没有必要添加。



### base64_decode

```cpp
fastring base64_decode(const void* s, size_t n);
fastring base64_decode(const char* s);
fastring base64_decode(const fastring& s);
fastring base64_decode(const std::string& s);
```

- `base64` 解码，如果输入的不是合理的 base64 编码的数据，解码将会失败，返回空字符串。




## nanoid

```cpp
1. fastring nanoid(int n=15);
2. fastring nanoid(const char* s, size_t len, int n);
3. fastring nanoid(const fastring& s, int n);
4. fastring nanoid(const std::string& s, int n);
5. fastring nanoid(const char* s, int n);
```

- 返回一个指定长度的随机字符串，该算法是 [ai/nanoid](https://github.com/ai/nanoid) 的 C++ 实现。
- 1, 使用默认的字母表，参数 n 指定随机字符串的长度，默认为 15。
- 2, 参数 s 与 len 指定一个字母表，参数 n 指定随机字符串的长度。
- 3-5, 参数 s 指定字母表，参数 n 指定随机字符串的长度。


- 示例

```cpp
auto s = nanoid();   // 15 byte id
auto x = nanoid(23); // 23 byte id

// 使用 0-9 与 a-f 中的字符生成长度为 8 的随机字符串
auto f = nanoid("0123456789abcdef", 8);
```




## url

### url_encode

```cpp
fastring url_encode(const void* s, size_t n);
fastring url_encode(const char* s);
fastring url_encode(const fastring& s);
fastring url_encode(const std::string& s);
```

- url 编码，保留字符 `!()*#$&'+,/:;=?@[]` 以及 `a-z A-Z 0-9 -_.~` 不会编码，所有其它的字符都会被编码。



### url_decode

```cpp
fastring url_decode(const void* s, size_t n);
fastring url_decode(const char* s);
fastring url_decode(const fastring& s);
fastring url_decode(const std::string& s);
```

- url 解码，如果输入的不是合理编码的 url，解码将会失败，返回空字符串。




## crc16

```cpp
uint16 crc16(const void* s, size_t n);
uint16 crc16(const char* s);
uint16 crc16(const fastring& s);
uint16 crc16(const std::string& s);
```

- 此函数计算字符串的 crc16 值，实现取自 [redis](https://github.com/antirez/redis/)，在实现 redis 集群客户端时会用到。
