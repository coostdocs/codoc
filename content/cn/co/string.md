---
weight: 3
title: "字符串"
---

## 头文件

```cpp
#include "co/string.h"
```

API 在 `co` 命名空间。


## co::string

`co::string` 是 coost 提供的字符串类，接口风格接近 `std::string`，但：

- 底层使用 [coost 内存分配器](../mem/)；
- 支持 `operator<<`，用于日志、print 等输出；
- 提供 `starts_with` / `ends_with` / `contains` / `trim` / `replace` 等便捷方法。


### 内部结构

```cpp
struct string {
    static const size_t npos = (size_t)-1;

    size_t _cap;   // 容量
    size_t _size;  // 长度
    char*  _p;     // 数据指针
};
```

- `_p` 由 coost 分配器管理。
- `_p` 可以为 `nullptr`（空字符串、未分配）。


### 构造函数

```cpp
constexpr string() noexcept;                       // 空字符串
explicit string(size_t cap) noexcept;              // 预分配 cap 字节
string(size_t n, char c) noexcept;                 // n 个字符 c
string(const void* s, size_t n) noexcept;          // 从 buffer 拷贝 n 字节
string(const char* s) noexcept;                    // 从 C 字符串
string(const string& s) noexcept;                  // 拷贝构造
string(const std::string& s) noexcept;             // 从 std::string
string(string&& s) noexcept;                       // 移动构造
```

说明：

- `string(size_t cap)` 只分配容量，`size() == 0`，`capacity() == cap`。
- `string(const void* s, size_t n)` 拷贝 `n` 字节，内部会分配 `n + 1` 字节。
- `string(const char* s)` 等价于 `string(s, s ? strlen(s) : 0)`，`s == nullptr` 时为空。


### 赋值

```cpp
string& operator=(string&& s) noexcept;
string& operator=(const char* s) noexcept;
string& operator=(const string& s) noexcept;
string& operator=(const std::string& s) noexcept;

string& assign(const void* s, size_t n) noexcept;
string& assign(size_t n, char c) noexcept;
template<typename S> string& assign(S&& s) noexcept;
```

- `assign(const void* s, size_t n)` 支持自引用：如果 `s` 在 `_p` 内部，会走 `memmove`。
- `assign(S&& s)` 转发到 `operator=`。


### 容量与大小

```cpp
char* data() noexcept;
const char* data() const noexcept;
size_t size() const noexcept;
bool empty() const noexcept;
size_t capacity() const noexcept;
void clear() noexcept;         // 只置 _size = 0
void zero_clear() noexcept;    // 把 _p[0.._size] 全部清零
const char* c_str() const noexcept;

void resize(size_t n) noexcept;    // 只改 size，不填充
void reserve(size_t n) noexcept;   // 保证 cap >= n
void reset() noexcept;             // 释放内存
void ensure(size_t n);             // 确保 cap > size + n
void shrink_to_fit() noexcept;     // 容量收缩到 size + 1
void swap(string& s) noexcept;
void swap(string&& s) noexcept;
```

- `clear()` 只设置 `_size = 0`，不释放内存。
- `zero_clear()` 将 `_p[0.._size]` 全部清零，并设置 `_size = 0`。
- `resize(n)` 只改 `_size`，扩展出来的内存不填 0。
- `c_str()` 保证 `_p[_size] == '\0'`，空字符串返回 `""`。
- `ensure(n)` 在容量不足时增长，增长策略是 `_cap += (_cap >> 1) + n + 1`。


### 元素访问

```cpp
char& back() noexcept;
const char& back() const noexcept;
char& front() noexcept;
const char& front() const noexcept;
char& operator[](size_t i) noexcept;
const char& operator[](size_t i) const noexcept;
```

- 这几个接口都不做边界检查，越界访问是未定义行为。
- 用户需要自己保证访问合法性：
  - `front() / back()` 要求字符串非空；
  - `operator[](i)` 要求 `i < size()`。


### 追加与修改

```cpp
string& append(char c) noexcept;
string& append(size_t n, char c) noexcept;
string& append(const void* s, size_t n) noexcept;
string& append(const char* s) noexcept;
string& append(const string& s) noexcept;
string& append(const std::string& s) noexcept;

string& append_nomchk(const void* p, size_t n) noexcept;
string& append_nomchk(const char* s) noexcept;

string& operator+=(char c) noexcept;
string& operator+=(const char* s) noexcept;
string& operator+=(const string& s) noexcept;
string& operator+=(const std::string& s) noexcept;

string& push_back(char c) noexcept;
char pop_back() noexcept;
```

- `append(const void* s, size_t n)` 支持自引用，会自动处理 `_p` 内部的情况。
- `append(const string& s)` 如果 `&s == this`，会做自追加：`_size <<= 1`。
- `append_nomchk` 不做自引用检查，调用方需保证 `p` 不在 `_p` 内部。


### operator<< 重载

`co::string` 支持流式拼接：

```cpp
string& operator<<(bool v) noexcept;
string& operator<<(char v) noexcept;
string& operator<<(signed char v) noexcept;
string& operator<<(unsigned char v) noexcept;
string& operator<<(short v) noexcept;
string& operator<<(unsigned short v) noexcept;
string& operator<<(int v) noexcept;
string& operator<<(unsigned int v) noexcept;
string& operator<<(long v) noexcept;
string& operator<<(unsigned long v) noexcept;
string& operator<<(long long v) noexcept;
string& operator<<(unsigned long long v) noexcept;
string& operator<<(double v) noexcept;
string& operator<<(float v) noexcept;
string& operator<<(const decimal& v) noexcept;
string& operator<<(const void* v) noexcept;
string& operator<<(std::nullptr_t) noexcept;
string& operator<<(const char* s) noexcept;
string& operator<<(const signed char* s) noexcept;
string& operator<<(const unsigned char* s) noexcept;
string& operator<<(const string& s) noexcept;
string& operator<<(const std::string& s) noexcept;
string& operator<<(const std::string_view& s) noexcept;
```

说明：

- `bool` 输出 `true` / `false`。
- 整型通过 `co::itoa` 输出。
- `double` 通过 `co::dtoa` 输出，默认最多 16 位有效小数。
- 指针输出为 `0x` + 十六进制。
- `nullptr` 输出 `0x0`。
- `decimal` 允许指定有效小数位数。

`decimal` 定义：

```cpp
struct decimal {
    constexpr decimal(double v, int n) noexcept : v(v), n(n) {}
    double v;
    int n; // significant decimal places
};
```

示例：

```cpp
co::string s;
s << "hello " << 23 << ' ' << 3.14 << ' ' << true;
s << co::decimal(3.14159, 2); // 保留两位有效小数位
```


### cat

```cpp
string& cat() noexcept;
template<typename X, typename ...V>
string& cat(X&& x, V&& ... v) noexcept;
```

- `cat(...)` 把参数依次 `<<` 到当前字符串。
- 等价于链式 `<<`，但更简洁。

示例：

```cpp
co::string s;
s.cat("hello ", 23, ' ', 3.14);
```


### 比较与查找

#### compare

```cpp
int compare(const char* s, size_t n) const noexcept;
int compare(const char* s) const noexcept;
int compare(const string& s) const noexcept;
int compare(const std::string& s) const noexcept;
```

返回：

- `< 0`：当前字符串小于 `s`；
- `0`：相等；
- `> 0`：当前字符串大于 `s`。

#### contains

```cpp
bool contains(char c) const noexcept;
bool contains(const char* s) const noexcept;
bool contains(const string& s) const noexcept;
bool contains(const std::string& s) const noexcept;
```

#### starts_with / ends_with

```cpp
bool starts_with(char c) const noexcept;
bool starts_with(const char* s, size_t n) const noexcept;
bool starts_with(const char* s) const noexcept;
bool starts_with(const string& s) const noexcept;
bool starts_with(const std::string& s) const noexcept;

bool ends_with(char c) const noexcept;
bool ends_with(const char* s, size_t n) const noexcept;
bool ends_with(const char* s) const noexcept;
bool ends_with(const string& s) const noexcept;
bool ends_with(const std::string& s) const noexcept;
```

#### find

```cpp
size_t find(char c) const noexcept;
size_t find(char c, size_t pos) const noexcept;
size_t find(char c, size_t pos, size_t len) const noexcept;

size_t find(const char* s) const noexcept;
size_t find(const char* s, size_t pos, size_t n) const noexcept;
size_t find(const char* s, size_t pos) const noexcept;
size_t find(const string& s, size_t pos=0) const noexcept;
size_t find(const std::string& s, size_t pos=0) const noexcept;
```

#### ifind（忽略大小写）

```cpp
size_t ifind(const char* s) const noexcept;
size_t ifind(const char* s, size_t pos, size_t n) const noexcept;
size_t ifind(const char* s, size_t pos) const noexcept;
size_t ifind(const string& s, size_t pos=0) const noexcept;
size_t ifind(const std::string& s, size_t pos=0) const noexcept;
size_t ifind(char c, size_t pos=0) const noexcept;
```

#### rfind

```cpp
size_t rfind(char c) const noexcept;
size_t rfind(char c, size_t pos) const noexcept;
size_t rfind(const char* s) const noexcept;
size_t rfind(const char* s, size_t pos, size_t n) const noexcept;
size_t rfind(const char* s, size_t pos) const noexcept;
size_t rfind(const string& s, size_t pos=npos) const noexcept;
size_t rfind(const std::string& s, size_t pos=npos) const noexcept;
```

#### find_first_of / find_first_not_of

```cpp
size_t find_first_of(const char* s, size_t pos, size_t n) const noexcept;
size_t find_first_of(const char* s, size_t pos=0) const noexcept;
size_t find_first_of(const string& s, size_t pos=0) const noexcept;
size_t find_first_of(const std::string& s, size_t pos=0) const noexcept;

size_t find_first_not_of(const char* s, size_t pos, size_t n) const noexcept;
size_t find_first_not_of(const char* s, size_t pos=0) const noexcept;
size_t find_first_not_of(const string& s, size_t pos=0) const noexcept;
size_t find_first_not_of(const std::string& s, size_t pos=0) const noexcept;
size_t find_first_not_of(char c, size_t pos=0) const noexcept;
```

#### find_last_of / find_last_not_of

```cpp
size_t find_last_of(const char* s, size_t pos, size_t n) const noexcept;
size_t find_last_of(const char* s, size_t pos=npos) const noexcept;
size_t find_last_of(const string& s, size_t pos=npos) const noexcept;
size_t find_last_of(const std::string& s, size_t pos=npos) const noexcept;

size_t find_last_not_of(const char* s, size_t pos, size_t n) const noexcept;
size_t find_last_not_of(const char* s, size_t pos=npos) const noexcept;
size_t find_last_not_of(const string& s, size_t pos=npos) const noexcept;
size_t find_last_not_of(const std::string& s, size_t pos=npos) const noexcept;
size_t find_last_not_of(char c, size_t pos=npos) const noexcept;
```

所有查找失败时返回 `co::string::npos`。


### 匹配、大小写、子串

```cpp
// * 匹配任意字符，? 匹配单个字符
bool match(const char* pattern) const noexcept;

// 将字符串转换为小写/大写
string& tolower() noexcept;
string& toupper() noexcept;

// 返回字符串的小写/大写拷贝
string lower() const noexcept;
string upper() const noexcept;

string substr(size_t pos) const noexcept;
string substr(size_t pos, size_t len) const noexcept;
```


### 转义与反转义

```cpp
// 转义字符：'"'、'\\'、'\0'、'\r'、'\n'、'\t'、'\a'、'\b'、'\f'、'\v'
string& escape() noexcept;
string& unescape() noexcept;
```


### 移除、修剪、替换

```cpp
string& remove_outer(size_t n) noexcept;        // 去掉首尾各 n 个字符
string& remove_prefix(size_t n) noexcept;       // 去掉前缀 n 个字符
string& remove_suffix(size_t n) noexcept;       // 去掉后缀 n 个字符

// 去掉前缀 s 
string& remove_prefix(const char* s, size_t n) noexcept;
string& remove_prefix(const char* s) noexcept;
string& remove_prefix(const string& s) noexcept;
string& remove_prefix(const std::string& s) noexcept;

// 去掉后缀 s
string& remove_suffix(const char* s, size_t n) noexcept;
string& remove_suffix(const char* s) noexcept;
string& remove_suffix(const string& s) noexcept;
string& remove_suffix(const std::string& s) noexcept;

// 去掉字符串两边的字符 c
string& trim(char c) noexcept;
string& trim_left(char c) noexcept;
string& trim_right(char c) noexcept;

// 去掉字符串两边在 s 中的字符
string& trim(const char* s=" \t\r\n") noexcept;
string& trim_left(const char* s=" \t\r\n") noexcept;
string& trim_right(const char* s=" \t\r\n") noexcept;

// @t: 最多替换次数，0 表示不限制
string& replace(const char* sub, size_t n, const char* to, size_t m, size_t t=0) noexcept;
string& replace(const char* sub, const char* to, size_t t=0) noexcept;
string& replace(const string& sub, const string& to, size_t t=0) noexcept;
```


## 数值转换函数

### itoa / utoh / ptoh / dtoa

```cpp
// 整数转十进制字符串
template<typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
inline int itoa(T v, char* buf, uint32 buf_size);

// 无符号整数转十六进制（带 0x 前缀）
template<typename T, typename = std::enable_if_t<std::is_integral_v<T> && std::is_unsigned_v<T>>>
inline int utoh(T v, char* buf, uint32 buf_size);

// 指针转十六进制（带 0x 前缀）
inline int ptoh(const void* v, char* buf, uint32 buf_size);

// double 转字符串，buf 长度至少为 25
inline int dtoa(double v, char* buf, int mdp=324);
```

### to_string

```cpp
inline string to_string(bool v) noexcept;
inline string to_string(int v) noexcept;
inline string to_string(unsigned int v) noexcept;
inline string to_string(long v) noexcept;
inline string to_string(unsigned long v) noexcept;
inline string to_string(long long v) noexcept;
inline string to_string(unsigned long long v) noexcept;
inline string to_string(double v) noexcept;
inline string to_string(float v) noexcept;
```

### 字符串转数值

```cpp
int32  stoi32(const char* s, int* err=nullptr) noexcept;
int64  stoi64(const char* s, int* err=nullptr) noexcept;
uint32 stou32(const char* s, int* err=nullptr) noexcept;
uint64 stou64(const char* s, int* err=nullptr) noexcept;
int    stoi  (const char* s, int* err=nullptr) noexcept;
bool   stob  (const char* s, int* err=nullptr) noexcept;
double stod  (const char* s, int* err=nullptr) noexcept;
```

- 同时提供 `co::string` 和 `std::string` 重载。
- 失败时通过 `err` 返回错误码，可能的错误码包括 `ERANGE`、`EINVAL`。


## 内存辅助函数

```cpp
char* co::memrchr(const char* s, char c, size_t n);

// 返回 s 中第一次出现 p 的位置
char* co::memmem (const char* s, size_t n, const char* p, size_t m);

// 忽略大小写，返回 s 中第一次出现 p 的位置
char* co::memimem(const char* s, size_t n, const char* p, size_t m);

// 返回 s 中最后一次出现 p 的位置
char* co::memrmem(const char* s, size_t n, const char* p, size_t m);

// 比较长度分别为 n 和 m 的两段内存
// 长度不等时返回 -1 或 1
inline int co::memcmp(const char* s, size_t n, const char* p, size_t m);
```


## 全局便捷函数

### split

```cpp
vector<string> split(const char* s, size_t n, char c, size_t t=0);
vector<string> split(const char* s, size_t n, const char* c, size_t m, size_t t=0);

inline vector<string> split(const char* s, char c, size_t t=0);
inline vector<string> split(const string& s, char c, size_t t=0);
inline vector<string> split(const char* s, const char* c, size_t t=0);
inline vector<string> split(const string& s, const char* c, size_t t=0);
```

- `t` 是最大分割次数，`t = 0` 表示不限制。

示例：

```cpp
co::split("|x|y|", '|');    // -> [ "", "x", "y" ]
co::split("xooy", 'o');     // -> [ "x", "", "y" ]
co::split("xooy", 'o', 1);  // -> [ "x", "oy" ]
```

### replace

```cpp
string replace(
    const char* s, size_t n,
    const char* sub, size_t m,
    const char* to, size_t l,
    size_t t=0
) noexcept;

inline string replace(
    const char* s, const char* sub, const char* to, size_t t=0) noexcept {
    return replace(s, strlen(s), sub, strlen(sub), to, strlen(to), t);
}
```

### remove_outer / remove_prefix / remove_suffix

```cpp
inline string remove_outer(const char* s, size_t n) noexcept;
inline string remove_outer(const string& s, size_t n) noexcept;
inline string remove_prefix(const char* s, size_t n) noexcept;
inline string remove_prefix(const string& s, size_t n) noexcept;
inline string remove_suffix(const char* s, size_t n) noexcept;
inline string remove_suffix(const string& s, size_t n) noexcept;
inline string remove_prefix(const char* s, const char* c) noexcept;
inline string remove_prefix(const string& s, const char* c) noexcept;
inline string remove_suffix(const char* s, const char* c) noexcept;
inline string remove_suffix(const string& s, const char* c) noexcept;
```

### trim / trim_left / trim_right

```cpp
inline string trim(const char* s, const char* c=" \t\r\n") noexcept;
inline string trim(const string& s, const char* c=" \t\r\n") noexcept;
inline string trim(const char* s, char c) noexcept;
inline string trim(const string& s, char c) noexcept;

inline string trim_left(const char* s, const char* c=" \t\r\n") noexcept;
inline string trim_left(const string& s, const char* c=" \t\r\n") noexcept;
inline string trim_left(const char* s, char c) noexcept;
inline string trim_left(const string& s, char c) noexcept;

inline string trim_right(const char* s, const char* c=" \t\r\n") noexcept;
inline string trim_right(const string& s, const char* c=" \t\r\n") noexcept;
inline string trim_right(const char* s, char c) noexcept;
inline string trim_right(const string& s, char c) noexcept;
```


## operator+ 与比较运算符

```cpp
inline co::string operator+(const co::string& a, char b) noexcept;
inline co::string operator+(char a, const co::string& b) noexcept;
inline co::string operator+(const co::string& a, const co::string& b) noexcept;
inline co::string operator+(const co::string& a, const std::string& b) noexcept;
inline co::string operator+(const std::string& a, const co::string& b) noexcept;
inline co::string operator+(const co::string& a, const char* b) noexcept;
inline co::string operator+(const char* a, const co::string& b) noexcept;
```

比较运算符支持：

- `co::string` 与 `co::string`
- `co::string` 与 `std::string`
- `co::string` 与 `const char*`

支持 `==`、`!=`、`<`、`>`、`<=`、`>=`，只比较内容，不关心底层实现。


## std::hash 特化

```cpp
namespace std {

template<>
struct hash<co::string> {
    size_t operator()(const co::string& s) const noexcept {
        return co::murmur_hash(s.data(), s.size());
    }
};

} // std
```

- `co::string` 可以直接用于 `std::unordered_map` / `std::unordered_set`。
- 哈希基于 `co::murmur_hash`。


## co::vector 别名

```cpp
template<class T, class Alloc = co::stl_allocator<T>>
using vector = std::vector<T, Alloc>;
```

- `co::vector<T>` 是使用 coost 分配器的 `std::vector`。
- `co::split` 返回 `co::vector<co::string>`。


## 与 std::ostream 的互操作

```cpp
inline std::ostream& operator<<(std::ostream& os, const co::string& s) noexcept {
    return os.write(s.data(), s.size());
}
```


## 示例

```cpp
#include "co/string.h"
#include "co/print.h"

int main() {
    co::string s;
    s << "hello " << 23 << ' ' << 3.14 << ' ' << true;
    co::println("s = ", s);

    co::string t = s;
    t.tolower();
    co::println("t = ", t);

    co::string u = co::trim("  hello  ", " ");
    co::println("u = ", u);

    auto v = co::split("a,b,c", ',');
    for (const auto& x : v) {
        co::println("part = ", x);
    }

    co::println("contains hello: ", s.contains("hello"));
    co::println("starts_with hello: ", s.starts_with("hello"));

    int err = 0;
    int x = co::stoi("123", &err);
    co::println("stoi = ", x, ", err = ", err);

    return 0;
}
```
