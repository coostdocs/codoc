---
weight: 6
title: "String Utility"
---

include: [co/str.h](https://github.com/idealvin/coost/tree/master/include/co/str.h).


## String operations


### str::cat

```cpp
template <typename ...X>
inline fastring cat(X&& ... x);
```

- Added in v2.0.3. Concatenate any number of elements to make a string.

- 示例

```cpp
str::cat("hello", ' ', 23);   // -> "hello 23"
str::cat("xx", 3.14, "true"); // -> "xx3.14true"
```



### str::replace

```cpp
fastring replace(const char* s, size_t n, const char* sub, size_t m, const char* to, size_t l, size_t t=0);
fastring replace(const char* s, size_t n, const char* sub, const char* to, size_t t=0);

fastring replace(const char* s, const char* sub, const char* to, size_t t=0);
fastring replace(const fastring& s, const char* sub, const char* to, size_t t=0);
fastring replace(const std::string& s, const char* sub, const char* to, size_t t=0);
```

- Replace substring `sub` in string `s` with `to`, `n` is the length of `s`, `m` is the length of `sub`, and `l` is the length of `to`; `t` is the maximum number of replacements, the default is 0 for unlimited.
- The content of the original string `s` remains unchanged, and the new string after replacement is returned.

- Example

```cpp
str::replace("xooxoox", "oo", "ee");    // -> "xeexeex"
str::replace("xooxoox", "oo", "ee", 1); // -> "xeexoox"
```



### str::split

```cpp
co::vector<fastring> split(const char* s, size_t n, char c, size_t t=0);
co::vector<fastring> split(const char* s, size_t n, const char* c, size_t m, size_t t=0);

co::vector<fastring> split(const char* s, char c, size_t t=0);
co::vector<fastring> split(const fastring& s, char c, size_t t=0);
co::vector<fastring> split(const std::string& s, char c, size_t t=0);

co::vector<fastring> split(const char* s, const char* c, size_t t=0);
co::vector<fastring> split(const fastring& s, const char* c, size_t t=0);
co::vector<fastring> split(const std::string& s, const char* c, size_t t=0);
```

- Split string `s` into several substrings by delimiter `c`. `n` is the length of `s`, `m` is the length of `c`, `t` is the maximum number of splits, and 0 means unlimited.
- The content of the original string `s` remains unchanged.

- Example

```cpp
str::split("x y z", '');   // -> ["x", "y", "z"]
str::split("|x|y|",'|');   // -> ["", "x", "y"]
str::split("xooy", "oo");  // -> ["x", "y"]
str::split("xooy",'o');    // -> ["x", "", "y"]
str::split("xooy",'o', 1); // -> ["x", "oy"]
```



### str::strip

```cpp
template<typename ...X>
inline fastring strip(X&& ...x) {
    return trim(std::forward<X>(x)...);
}
```

- Equivalent to [str::trim](#strtrim)。



### str::trim

```cpp
fastring trim(const char* s, const char* c=" \t\r\n", char d='b');
fastring trim(const char* s, char c, char d='b');
fastring trim(const fastring& s, const char* c=" \t\r\n", char d='b');
fastring trim(const fastring& s, char c, char d='b');
```

- Trim the string, removing character `c` or characters in string `c` from the left or right side of string `s`. The parameter `d` is the direction, 'l' or 'L' means left, 'r' or 'R' means right, default is 'b' for both sides.
- The content of the original string `s` remains unchanged.

- Example

```cpp
str::trim(" xx\r\n");         // -> "xx"
str::trim("abxxa", "ab");     // -> "xx"
str::trim("abxxa", "ab",'l'); // -> "xxa"
str::trim("abxxa", "ab",'r'); // -> "abxx"
```



## Conversion

### str::to_bool

```cpp
bool to_bool(const char* s);
bool to_bool(const fastring& s);
bool to_bool(const std::string& s);
```

- This function converts a string to bool type.
- When `s` is equal to "0" or "false", false is returned; when `s` is equal to "1" or "true", true is returned.
- If the conversion is successful, the error code is 0. Otherwise, the error code is set to EINVAL, and false is returned. Call [co::error()](../error/#coerror) to get the error code.

- Example

```cpp
bool b = str::to_bool("true");  // b = true
bool x = str::to_bool("false"); // x = false
```



### str::to_double

```cpp
double to_double(const char* s);
double to_double(const fastring& s);
double to_double(const std::string& s);
```

- This function converts a string to double type.
- If the conversion is successful, the error code is 0. Otherwise, the error code is set to ERANGE or EINVAL, and 0 is returned. Call [co::error()](../error/#coerror) to get the error code.


- Example

```cpp
double x = str::to_double("3.14"); // x = 3.14
```



### str::to_int

```cpp
int32 to_int32(const char* s);
int32 to_int32(const fastring& s);
int32 to_int32(const std::string& s);
int64 to_int64(const char* s);
int64 to_int64(const fastring& s);
int64 to_int64(const std::string& s);
uint32 to_uint32(const char* s);
uint32 to_uint32(const fastring& s);
uint32 to_uint32(const std::string& s);
uint64 to_uint64(const char* s);
uint64 to_uint64(const fastring& s);
uint64 to_uint64(const std::string& s);
```

- These functions convert a string to integer types.
- The parameter `s` can take one unit `k, m, g, t, p` at the end, which is not case sensitive.
- If the conversion is successful, the error code is 0. Otherwise, the error code is set to ERANGE or EINVAL, and 0 is returned. Call [co::error()](../error/#coerror) to get the error code.

- Example

```cpp
int32 i32;
int64 i64;
uint32 u32;
uint64 u64;

i32 = str::to_int32("-23");  // -23
u32 = str::to_uint32("4k");  // 4096
i64 = str::to_int32("8M");   // 8 << 20
i64 = str::to_int64("8T");   // 8ULL << 40
u64 = str::to_int64("1P");   // 1ULL << 50

i32 = str::to_int32("8g");
LOG << (i32 == 0);
LOG << (errno == ERANGE);

i32 = str::to_int32("abx");
LOG << (i32 == 0);
LOG << (errno == EINVAL);
```



### str::from

```cpp
template<typename T>
inline fastring from(T t);
```

- This function converts built-in types to a string.
- T can be any built-in type, such as bool, int, double, void*, etc.

- Example

```cpp
fastring s;
s = str::from(true); // -> "true"
s = str::from(23);   // -> "23"
s = str::from(3.14); // -> "3.14"
```



### str::dbg

```cpp
template<typename T> fastring dbg(const co::vector<T>& v);
template<typename T> fastring dbg(const std::vector<T>& v);
template<typename T> fastring dbg(const co::list<T>& v);
template<typename T> fastring dbg(const std::list<T>& v);
template<typename T> fastring dbg(const co::deque<T>& v);
template<typename T> fastring dbg(const std::deque<T>& v);

template<typename T> fastring dbg(const co::set<T>& v);
template<typename T> fastring dbg(const std::set<T>& v);
template<typename T> fastring dbg(const co::hash_set<T>& v);
template<typename T> fastring dbg(const std::unordered_set<T>& v);

template<typename K, typename V>
fastring dbg(const co::map<K, V>& v);

template<typename K, typename V>
fastring dbg(const std::map<K, V>& v);

template<typename K, typename V>
fastring dbg(const co::hash_map<K, V>& v);

template<typename K, typename V>
fastring dbg(const std::unordered_map<K, V>& v);

template<typename K, typename V>
fastring dbg(const co::lru_map<K, V>& v);
```

- This function converts a container to a debug string, which is generally for printing logs.


- Example

```cpp
std::vector<int> v {1, 2, 3 };
std::set<int> s {1, 2, 3 };
std::map<int, int> m {{1, 1}, {2, 2} };
str::dbg(v); // -> "[1,2,3]"
str::dbg(s); // -> "{1,2,3}"
str::dbg(m); // -> "{1:1,2:2}

std::vector<std::vector<int>> vv = {
    {1, 2, 3},
    {6, 7, 8},
};
str::dbg(vv); // -> "[[1,2,3],[6,7,8]]"
```

{{< hint info >}}
Since v3.0.1, `str::dbg` supports multiple nested containers.
{{< /hint >}}
