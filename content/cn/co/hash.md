---
weight: 20
title: "Hash"
---


include: [co/hash.h](https://github.com/idealvin/co/blob/master/include/co/hash.h).


## Hash




### hash32
```cpp
uint32 hash32(const char* s);
uint32 hash32(const fastring& s)
uint32 hash32(const std::string& s);
uint32 hash32(const void* s, size_t n);
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





### md5sum
```cpp
fastring md5sum(const void* s, size_t n);
fastring md5sum(const char* s);
fastring md5sum(const fastring& s);
fastring md5sum(const std::string& s);
```

- 此函数计算字符串的 md5 值，返回长度为 32 字节、仅含十六进制字符的字符串。





### crc16
```cpp
uint16_t crc16(const void* s, size_t n);
uint16_t crc16(const char* s);
uint16_t crc16(const fastring& s);
uint16_t crc16(const std::string& s);
```

- 此函数计算字符串的 crc16 值，实现取自 [redis](https://github.com/antirez/redis/)，在实现 redis 集群客户端时会用到。





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





### url_encode
```cpp
fastring url_encode(const void* s, size_t n);
fastring url_encode(const char* s);
fastring url_encode(const fastring& s);
fastring url_encode(const std::string& s);
```

- url 编码，保留字符 `!()*#$&'+,/:;=?@[]`  以及 `a-z A-Z 0-9 -_.~` 不会编码，所有其它的字符都会被编码。





### url_decode
```cpp
fastring url_decode(const void* s, size_t n);
fastring url_decode(const char* s);
fastring url_decode(const fastring& s);
fastring url_decode(const std::string& s);
```

- url 解码，如果输入的不是合理编码的 url，解码将会失败，返回空字符串。



