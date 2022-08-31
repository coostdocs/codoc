---
weight: 1
title: "基本定义"
---

include: [co/def.h](https://github.com/idealvin/co/blob/master/include/co/def.h).


## typedefs


### 定长整数类型

`def.h` 定义了如下的 8 种整数类型：

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

这些类型在不同平台的长度是一致的，不存在可移植性问题。[Google Code Style](https://google.github.io/styleguide/cppguide.html#Integer_Types) 建议除了 int，不要使用 short, long, long long 等内置整数类型。




## macros


### 整型最大、最小值

```cpp
MAX_UINT8  MAX_UINT16  MAX_UINT32  MAX_UINT64
MAX_INT8   MAX_INT16   MAX_INT32   MAX_INT64
MIN_INT8   MIN_INT16   MIN_INT32   MIN_INT64
```

这些宏分别表示 8 种整数类型的最大、最小值。



### DISALLOW_COPY_AND_ASSIGN

这个宏用于禁止 C++ 类中的拷贝构造函数与赋值操作。

- 示例

```cpp
class T {
  public:
    T();
    DISALLOW_COPY_AND_ASSIGN(T);
};
```



### __arch64, __arch32

64位系统上，`__arch64` 定义为 1；32位系统上，`__arch32` 定义为 1。

- 示例

```cpp
#if __arch64
inline size_t murmur_hash(const void* s, size_t n) {
    return murmur_hash64(s, n, 0);
}
#else
inline size_t murmur_hash(const void* s, size_t n) {
    return murmur_hash32(s, n, 0);
}
#endif
```



### __forceinline

[__forceinline](https://docs.microsoft.com/en-us/cpp/cpp/inline-functions-cpp?view=vs-2019#inline-__inline-and-__forceinline) 是 VS 中的关键字，放在函数定义开头，强制内联函数，linux 与 mac 平台用下面的宏模拟：

```cpp
#define __forceinline __attribute__((always_inline))
```



### __thread

[__thread](https://gcc.gnu.org/onlinedocs/gcc-4.7.4/gcc/Thread-Local.html) 是 gcc/clang 中的关键字，用于支持 [TLS](https://wiki.osdev.org/Thread_Local_Storage)，windows 平台用下面的宏模拟：

```cpp
#define __thread __declspec(thread)
```

- 示例

```cpp
// get id of the current thread
__forceinline unsigned int gettid() {
    static __thread unsigned int id = 0;
    if (id != 0) return id;
    return id = __gettid();
}
```



### unlikely

这个宏用于分支选择优化，仅支持 gcc/clang。

- 示例

```cpp
// 与 if (v == 0) 逻辑上等价，但提示编译器 v == 0 的可能性较小
if (unlikey(v == 0)) {
    cout << "v == 0" << endl;
}
```



### `__fname__` 与 `__fnlen__`

`__fname__` 宏用于在编译期获取 `__FILE__` 中的文件名部分，不包含文件路径。`__fnlen__` 宏则用于在编译期获取 `__fname__` 的长度，不包含 `\0`。

- 示例

```cpp
// test/xx.cc
static_assert(__fname__[0] == 'x', "");
static_assert(__fnlen__ == 5, "");
```
