---
weight: 20
title: "Hash"
---

include: [co/hash.h](https://github.com/idealvin/co/blob/master/include/co/hash.h).


## Hash

### hash32

```cpp
uint32 hash32(const void* s, size_t n);
uint32 hash32(const char* s);
uint32 hash32(const fastring& s)
uint32 hash32(const std::string& s);
```

- This function returns a 32-bit murmur hash value.
- When s is a pointer, it is generally required to be `sizeof(void*)` byte aligned.



### hash64

```cpp
uint64 hash64(const void* s, size_t n);
uint64 hash64(const char* s);
uint64 hash64(const fastring& s);
uint64 hash64(const std::string& s);
```

- This function returns a 64-bit murmur hash value.
- When s is a pointer, it is generally required to be 8-byte aligned.



### murmur_hash

```cpp
size_t murmur_hash(const void* s, size_t n);
```

- This function returns a hash value of `size_t` type. This value is 64-bit on 64-bit platform and 32-bit on 32-bit platform.
- The parameter s is generally required to be `sizeof(void*)` byte-aligned.




## md5

### md5digest

```cpp
void md5digest(const void* s, size_t n, char res[16]);
fastring md5digest(const void* s, size_t n);
fastring md5digest(const char* s);
fastring md5digest(const fastring& s);
fastring md5digest(const std::string& s);
```

- This function calculates the md5 of a string and returns a 16-byte binary string.



### md5sum

```cpp
void md5sum(const void* s, size_t n, char res[32]);
fastring md5sum(const void* s, size_t n);
fastring md5sum(const char* s);
fastring md5sum(const fastring& s);
fastring md5sum(const std::string& s);
```

- This function calculates the md5 of a string and returns a 32-byte string containing only hexadecimal characters(0-9,a-f).



### Lower level APIs

```cpp
void md5_init(md5_ctx_t* ctx);
void md5_update(md5_ctx_t* ctx, const void* s, size_t n);
void md5_final(md5_ctx_t* ctx, uint8 res[16]);
```

- The above 3 APIs can be used to calculate md5 incrementally.


- Example

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

- This function calculates the sha256 of a string and returns a 32-byte binary string.



### sha256sum

```cpp
void sha256sum(const void* s, size_t n, char res[64]);
fastring sha256sum(const void* s, size_t n);
fastring sha256sum(const char* s);
fastring sha256sum(const fastring& s);
fastring sha256sum(const std::string& s);
```

- This function calculates the sha256 of a string and returns a 64-byte string containing only hexadecimal characters(0-9,a-f).



### Lower level APIs

```cpp
void sha256_init(sha256_ctx_t* ctx);
void sha256_update(sha256_ctx_t* ctx, const void* s, size_t n);
void sha256_final(sha256_ctx_t* ctx, uint8 res[32]);
```

- The above 3 APIs can be used to calculate sha256 incrementally like that in md5.




## base64

### base64_encode

```cpp
fastring base64_encode(const void* s, size_t n);
fastring base64_encode(const char* s);
fastring base64_encode(const fastring& s);
fastring base64_encode(const std::string& s);
```

- `base64` encoding, `\r\n` is not added in this implementation.



### base64_decode

```cpp
fastring base64_decode(const void* s, size_t n);
fastring base64_decode(const char* s);
fastring base64_decode(const fastring& s);
fastring base64_decode(const std::string& s);
```

- `base64` decoding, if the input is not valid base64-encoded data, the decoding will fail and an empty string will be returned.




## nanoid

```cpp
1. fastring nanoid(int n=15);
2. fastring nanoid(const char* s, size_t len, int n);
3. fastring nanoid(const fastring& s, int n);
4. fastring nanoid(const std::string& s, int n);
5. fastring nanoid(const char* s, int n);
```

- Returns a random string of the specified length. It is a C++ implementation of [ai/nanoid](https://github.com/ai/nanoid).
- 1, use the default alphabet, the parameter n specifies the length of the random string, the default is 15.
- 2, the parameters s and len specify an alphabet, and the parameter n specifies the length of the random string.
- 3-5, parameter s specifies the alphabet, and parameter n specifies the length of the random string.


- Example

````cpp
auto s = nanoid();   // 15 byte id
auto x = nanoid(23); // 23 byte id

// Generate a 8-byte random string using characters in 0-9,a-f
auto f = nanoid("0123456789abcdef", 8);
````




## url

### url_encode

```cpp
fastring url_encode(const void* s, size_t n);
fastring url_encode(const char* s);
fastring url_encode(const fastring& s);
fastring url_encode(const std::string& s);
```

- url encoding, reserved characters `!()*#$&'+,/:;=?@[]` and `a-z A-Z 0-9 -_.~` will not be encoded, all other characters will be encoded.



### url_decode

```cpp
fastring url_decode(const void* s, size_t n);
fastring url_decode(const char* s);
fastring url_decode(const fastring& s);
fastring url_decode(const std::string& s);
```

- url decoding, if the input is not a reasonably encoded url, the decoding will fail and an empty string will be returned.




## crc16

```cpp
uint16_t crc16(const void* s, size_t n);
uint16_t crc16(const char* s);
uint16_t crc16(const fastring& s);
uint16_t crc16(const std::string& s);
```

- This function calculates the crc16 value of a string. The implementation is taken from [redis](https://github.com/antirez/redis/) and will be used when implementing the redis cluster client.
