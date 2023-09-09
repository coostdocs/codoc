---
weight: 6
title: "string utility"
---

include: [co/str.h](https://github.com/idealvin/coost/tree/master/include/co/str.h).


## 字符串操作


### str::cat

```cpp
template<typename ...X>
inline fastring cat(X&& ...x);
```

- v2.0.3 新增，将任意数量的元素连接为一个字符串。

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

- 将字符串 s 中的子串 sub 替换为 to，n 是 s 的长度，m 是 sub 的长度，l 是 to 的长度；t 是最大替换次数，默认为 0 不限次数。
- 原字符串 s 内容保持不变，返回替换后的新字符串。

- 示例

```cpp
str::replace("xooxoox", "oo", "ee");     // -> "xeexeex"
str::replace("xooxoox", "oo", "ee", 1);  // -> "xeexoox"
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

- 将字符串 s 按分隔符 c 切分成若干个子串，返回切分后的结果。n 是 s 的长度，m 是 c 的长度，t 是最大切分次数，0 表示不限。
- 原字符串 s 内容保持不变，返回切分后的结果。

- 示例

```cpp
str::split("x y z", ' ');    // ->  [ "x", "y", "z" ]
str::split("|x|y|", '|');    // ->  [ "", "x", "y" ]
str::split("xooy", "oo");    // ->  [ "x", "y"]
str::split("xooy", 'o');     // ->  [ "x", "", "y" ]
str::split("xooy", 'o', 1);  // ->  [ "x", "oy" ]
```



### str::strip

```cpp
template<typename ...X>
inline fastring strip(X&& ...x) {
    return trim(std::forward<X>(x)...);
}
```

- 与 [str::trim](#strtrim) 等价。



### str::trim

```cpp
fastring trim(const char* s, const char* c=" \t\r\n", char d='b');
fastring trim(const char* s, char c, char d='b');
fastring trim(const fastring& s, const char* c=" \t\r\n", char d='b');
fastring trim(const fastring& s, char c, char d='b');
```

- 修剪字符串，从字符串 s 两边去掉字符 c 或字符串 c 中的字符。参数 d 是方向，'l' 或 'L' 表示左边，'r' 或 'R' 表示右边，默认为 'b' 表示两边。
- 原字符串 s 内容保持不变，返回修剪后的新字符串。

- 示例

```cpp
str::trim(" xx\r\n");           // -> "xx"
str::trim("abxxa", "ab");       // -> "xx"
str::trim("abxxa", "ab", 'l');  // -> "xxa"
str::trim("abxxa", "ab", 'r');  // -> "abxx"
```




## 类型转换

### str::to_bool

```cpp
bool to_bool(const char* s);
bool to_bool(const fastring& s);
bool to_bool(const std::string& s);
```

- 将字符串转换为 bool 类型。
- 当 s 等于 "0" 或 "false"，返回 false；当 s 等于 "1" 或 "true"，返回 true。
- 此函数转换成功时 error code 为 0，转换失败时返回 false，并将 error code 设置为 EINVAL，可以调用 [co::error()](../error/#coerror) 获取错误码。

- 示例

```cpp
bool b = str::to_bool("true");   // b = true
bool x = str::to_bool("false");  // x = false
```



### str::to_double

```cpp
double to_double(const char* s);
double to_double(const fastring& s);
double to_double(const std::string& s);
```

- 将字符串转换为 double 类型。
- 此函数转换成功时 error code 为 0，转换失败时返回 0，并设置 error code 为 ERANGE 或 EINVAL，可以调用 [co::error()](../error/#coerror) 获取错误码。

- 示例

```cpp
double x = str::to_double("3.14");  // x = 3.14
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

- 将字符串转换为整数类型。
- 参数 s 末尾可以带一个单位 `k, m, g, t, p`，不区分大小写。
- 这些函数转换成功时 error code 为 0，转换失败时返回 0，并设置 error code 为 ERANGE 或 EINVAL，可以调用 [co::error()](../error/#coerror) 获取错误码。

- 示例

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
LOG << (co::error() == ERANGE);

i32 = str::to_int32("abx");
LOG << (i32 == 0);
LOG << (co::error() == EINVAL);
```



### str::from

```cpp
template<typename T>
inline fastring from(T t);
```

- 此函数将内置类型转换为字符串。
- T 可以是任意内置类型，如 bool, int, double, void* 等等。

- 示例

```cpp
fastring s;
s = str::from(true);  // -> "true"
s = str::from(23);    // -> "23"
s = str::from(3.14);  // -> "3.14"
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

- 此函数将常用的容器类型转换成一个 debug 字符串，一般用于打印日志。

- 示例

```cpp
std::vector<int> v { 1, 2, 3 };
std::set<int> s { 1, 2, 3 };
std::map<int, int> m { {1, 1}, {2, 2} };
str::dbg(v);  // -> "[1,2,3]"
str::dbg(s);  // -> "{1,2,3}"
str::dbg(m);  // -> "{1:1,2:2}

std::vector<std::vector<int>> vv = {
    {1, 2, 3},
    {6, 7, 8},
};
str::dbg(vv); // -> "[[1,2,3],[6,7,8]]"
```

{{< hint info >}}
从 v3.0.1 开始，`str::dbg` 支持多重嵌套的容器。
{{< /hint >}}
