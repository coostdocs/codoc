---
weight: 4
title: "字符串操作"
---

include: [co/str.h](https://github.com/idealvin/co/tree/master/include/co/str.h).

## 切分、修剪、替换




### str::split
```cpp
std::vector<fastring> split(const char* s, char c, uint32 n=0);
std::vector<fastring> split(const fastring& s, char c, uint32 n=0);
std::vector<fastring> split(const char* s, const char* c, uint32 n=0);
std::vector<fastring> split(const fastring& s, const char* c, uint32 n=0)；
```

- 此函数将字符串切分成若干个子串，原字符串保持不变，返回切分后的结果。
- 参数 s 是 C 字符串或 fastring，参数 c 是分隔符或 C 字符串，参数 n 是最大切分次数，0 或 -1 表示不限次数。
- 第 4 个版本中，s 不能包含 '\0'，因为内部实现中需要用 `strstr()` 搜索子串。



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
fastring strip(const char* s, const char* c=" \t\r\n", char d='b');
fastring strip(const fastring& s, const char* c=" \t\r\n", char d='b');
fastring strip(const char* s, char c, char d = 'b');
fastring strip(const fastring& s, char c, char d='b');
fastring strip(const fastring& s, const fastring& c, char d='b');
```

- 此函数去掉字符串左边或右边的字符，原字符串保持不变，返回 strip 后的结果。
- 参数 s 是 C 字符串或 fastring，参数 c 是要去掉的字符集，参数 d 是方向，'l' 或 'L' 表示左边，'r' 或 'R' 表示右边，默认为 'b' 表示两边。
- 第 1 个与第 2 个版本默认去掉字符串两边的空白字符。
- 第 3 个与第 4 个版本中，c 是单个字符。




- 示例
```cpp
str::strip(" xx\r\n");           // -> "xx"
str::strip("abxxa", "ab");       // -> "xx"
str::strip("abxxa", "ab", 'l');  // -> "xxa"
str::strip("abxxa", "ab", 'r');  // -> "abxx"
```




### str::replace
```cpp
fastring replace(const char* s, const char* sub, const char* to, uint32 n=0);
fastring replace(const fastring& s, const char* sub, const char* to, uint32 n=0);
```

- 此函数用于替换字符串中的子串，原字符串保持不变，返回替换后的结果。
- 参数 s 是 C 字符串或 fastring，参数 sub 是要替换的子串，参数 to 是替换后的子串，参数 n 是最大替换次数，0 或 -1 表示不限次数。
- 第 2 个版本中，s 不能包含 '\0'，因为内部实现中需要用 `strstr()` 搜索子串。



- 示例
```cpp
str::replace("xooxoox", "oo", "ee");     // -> "xeexeex"
str::replace("xooxoox", "oo", "ee", 1);  // -> "xeexoox"
```






## 字符串转换为内置类型




### str::to_bool
```cpp
bool to_bool(const char* s);
bool to_bool(const fastring& s);
bool to_bool(const std::string& s);
```

- 此函数将字符串转换为 bool 类型。
- 当 s 等于 "0" 或 "false"，返回 false；当 s 等于 "1" 或 "true"，返回 true。
- 此函数**转换失败时返回 false，并将 errno 设置为 EINVAL**。



- 示例
```cpp
bool b = str::to_bool("true");   // x = true
bool x = str::to_bool("false");  // x = false
```




### str::to_double
```cpp
double to_double(const char* s);
double to_double(const fastring& s);
double to_double(const std::string& s);
```

- 此函数将字符串转换为 double 类型。
- 此函数**转换失败时返回 0，并设置 errno 为 ERANGE 或 EINVAL**。




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

- 这些函数将字符串转换为整数类型。
- 参数 s 末尾可以带一个单位 `k, m, g, t, p`，不区分大小写。
- 这些函数**转换失败时返回 0，并设置 errno 为 ERANGE 或 EINVAL**。




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
LOG << (errno == ERANGE);

i32 = str::to_int32("abx");
LOG << (i32 == 0);
LOG << (errno == EINVAL);
```






## 内置类型转换为字符串




### str::from
```cpp
template<typename T>
inline fastring from(T t);
```

- 此函数将内置类型转换为字符串。
- T 可以是任意内置类型，如 bool, int, double, void* 等待。



- 示例
```cpp
fastring s;
s = str::from(true);  // -> "true"
s = str::from(23);    // -> "23"
s = str::from(3.14);  // -> "3.14"
```








## STL 容器转换成 debug string




### str::dbg
```cpp
template<typename T> fastring dbg(const std::vector<T>& v);
template<typename T> fastring dbg(const std::set<T>& v);
template<typename T> fastring dbg(const std::unordered_set<T>& v)
template<typename K, typename V> fastring dbg(const std::map<K, V>& v);
template<typename K, typename V> fastring dbg(const std::unordered_map<K, V>& v);
```

- 此函数将常用的容器类型转换成一个 debug 字符串，一般用于打印日志。
- 容器中的字符串类型，两边会加上 `"`，但不会对字符串内的双引号进行转义。



- 示例
```cpp
std::vector<int> v { 1, 2, 3 };
std::set<int> s { 1, 2, 3 };
std::map<int, int> m { {1, 1}, {2, 2} };
str::dbg(v);    // -> "[1,2,3]"
str::dbg(s);    // -> "{1,2,3}"
str::dbg(m);    // -> "{1:1,2:2}
```


