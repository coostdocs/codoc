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


- This function returns a 32-bit murmur hash value.
- When s is a pointer, it is generally required to be byte-aligned by `sizeof(void*)`.





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





### md5sum


```cpp
fastring md5sum(const void* s, size_t n);
fastring md5sum(const char* s);
fastring md5sum(const fastring& s);
fastring md5sum(const std::string& s);
```


- This function calculates the md5 value of a string and returns a 32-byte string containing only hexadecimal characters.





### crc16


```cpp
uint16_t crc16(const void* s, size_t n);
uint16_t crc16(const char* s);
uint16_t crc16(const fastring& s);
uint16_t crc16(const std::string& s);
```


- This function calculates the crc16 value of a string. The implementation is taken from [redis](https://github.com/antirez/redis/) and will be used when implementing the redis cluster client.





### base64_encode


```cpp
fastring base64_encode(const void* s, size_t n);
fastring base64_encode(const char* s);
fastring base64_encode(const fastring& s);
fastring base64_encode(const std::string& s);
```


- `base64` encoding, `\r\n` is not added in the implementation.





### base64_decode


```cpp
fastring base64_decode(const void* s, size_t n);
fastring base64_decode(const char* s);
fastring base64_decode(const fastring& s);
fastring base64_decode(const std::string& s);
```


- `base64` decoding, if the input is not reasonable base64-encoded data, the decoding will fail and an empty string will be returned.





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



