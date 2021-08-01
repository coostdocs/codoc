---
weight: 1
title: "Basic definitions"
---

include: [co/def.h](https://github.com/idealvin/co/blob/master/include/co/def.h).


## typedefs


### Fixed-length integer type


`def.h` defines the following 8 types of integers:
```cpp
typedef int8_t   int8;
typedef int16_t  int16;
typedef int32_t  int32;
typedef int64_t  int64;
typedef uint8_t  uint8;
typedef uint16_t uint16;
typedef uint32_t uint32;
typedef uint64_t uint64;
```
These types have the same length on different platforms, and there is no portability problem. [Google Code Style](https://google.github.io/styleguide/cppguide.html#Integer_Types) recommended not to use built-in integer types such as short, long, long long, etc.


## macros


### Maximum and minimum values of integer types
```cpp
MAX_UINT8  MAX_UINT16  MAX_UINT32  MAX_UINT64
MAX_INT8   MAX_INT16   MAX_INT32   MAX_INT64
MIN_INT8   MIN_INT16   MIN_INT32   MIN_INT64
```
These macros respectively defines the maximum and minimum values of the 8 integer types.


### DISALLOW_COPY_AND_ASSIGN


This macro is used to disable copy constructor and assignment operations in C++ classes. 


- Example
```cpp
class T {
  public:
    T();
    DISALLOW_COPY_AND_ASSIGN(T);
};
```


### __forceinline


[__forceinline](https://docs.microsoft.com/en-us/cpp/cpp/inline-functions-cpp?view=vs-2019#inline-__inline-and-__forceinline) is a keyword in VS. Linux and mac platforms use the following macro simulation:
```cpp
#define __forceinline __attribute__((always_inline))
```


### __thread


[__thread](https://gcc.gnu.org/onlinedocs/gcc-4.7.4/gcc/Thread-Local.html) is a keyword in gcc/clang to support [TLS](https://wiki.osdev.org/Thread_Local_Storage), the windows platform uses the following macro simulation:
```cpp
#define __thread __declspec(thread)
```


- Example
```cpp
// get id of the current thread
__forceinline unsigned int gettid() {
    static __thread unsigned int id = 0;
    if (id != 0) return id;
    return id = __gettid();
}
```


### unlikely


This macro is used for branch prediction optimization. It only supports gcc/clang.


- Example
```cpp
// It is logically equivalent to if (v == 0)
if (unlikey(v == 0)) {
    cout << "v == 0" << endl;
}
```


