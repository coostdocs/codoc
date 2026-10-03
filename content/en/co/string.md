---
weight: 3
title: "String"
---

## Header

```cpp
#include "co/string.h"
```

The API is in the `co` namespace.

## co::string

`co::string` is the string class provided by coost. Its interface style is close to `std::string`, but:

- It uses the [coost memory allocator](../mem/) underneath;
- It supports `operator<<`, used for output in logging, print, etc.;
- It provides convenient methods such as `starts_with` / `ends_with` / `contains` / `trim` / `replace`.

### Internal Structure

```cpp
struct string {
    static const size_t npos = (size_t)-1;

    size_t _cap;   // capacity
    size_t _size;  // length
    char*  _p;     // data pointer
};
```

- `_p` is managed by the coost allocator.
- `_p` can be `nullptr` (empty string, not allocated).

### Constructors

```cpp
constexpr string() noexcept;                       // empty string
explicit string(size_t cap) noexcept;              // pre-allocate cap bytes
string(size_t n, char c) noexcept;                 // n characters c
string(const void* s, size_t n) noexcept;          // copy n bytes from buffer
string(const char* s) noexcept;                    // from C string
string(const string& s) noexcept;                  // copy constructor
string(const std::string& s) noexcept;             // from std::string
string(string&& s) noexcept;                       // move constructor
```

Notes:

- `string(size_t cap)` only allocates capacity, `size() == 0`, `capacity() == cap`.
- `string(const void* s, size_t n)` copies `n` bytes, and internally allocates `n + 1` bytes.
- `string(const char* s)` is equivalent to `string(s, s ? strlen(s) : 0)`, and is empty when `s == nullptr`.

### Assignment

```cpp
string& operator=(string&& s) noexcept;
string& operator=(const char* s) noexcept;
string& operator=(const string& s) noexcept;
string& operator=(const std::string& s) noexcept;

string& assign(const void* s, size_t n) noexcept;
string& assign(size_t n, char c) noexcept;
template<typename S> string& assign(S&& s) noexcept;
```

- `assign(const void* s, size_t n)` supports self-reference: if `s` is inside `_p`, it uses `memmove`.
- `assign(S&& s)` forwards to `operator=`.

### Capacity and Size

```cpp
char* data() noexcept;
const char* data() const noexcept;
size_t size() const noexcept;
bool empty() const noexcept;
size_t capacity() const noexcept;
void clear() noexcept;         // only sets _size = 0
void zero_clear() noexcept;    // zeroes _p[0.._size]
const char* c_str() const noexcept;

void resize(size_t n) noexcept;    // only changes size, does not fill
void reserve(size_t n) noexcept;   // ensures cap >= n
void reset() noexcept;             // frees memory
void ensure(size_t n);             // ensures cap > size + n
void shrink_to_fit() noexcept;     // shrinks capacity to size + 1
void swap(string& s) noexcept;
void swap(string&& s) noexcept;
```

- `clear()` only sets `_size = 0`, it does not free memory.
- `zero_clear()` zeroes `_p[0.._size]` and sets `_size = 0`.
- `resize(n)` only changes `_size`; the extended memory is not filled with 0.
- `c_str()` guarantees `_p[_size] == '\0'`; an empty string returns `""`.
- `ensure(n)` grows when capacity is insufficient; the growth strategy is `_cap += (_cap >> 1) + n + 1`.

### Element Access

```cpp
char& back() noexcept;
const char& back() const noexcept;
char& front() noexcept;
const char& front() const noexcept;
char& operator[](size_t i) noexcept;
const char& operator[](size_t i) const noexcept;
```

- These interfaces do not perform bounds checking; out-of-bounds access is undefined behavior.
- Users need to ensure access validity themselves:
  - `front() / back()` require the string to be non-empty;
  - `operator[](i)` requires `i < size()`.

### Append and Modify

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

- `append(const void* s, size_t n)` supports self-reference and automatically handles the case inside `_p`.
- `append(const string& s)` does self-append if `&s == this`: `_size <<= 1`.
- `append_nomchk` does not perform self-reference checking; the caller must ensure `p` is not inside `_p`.

### `operator<<` Overloads

`co::string` supports streaming concatenation:

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

Notes:

- `bool` outputs `true` / `false`.
- Integers are output through `co::itoa`.
- `double` is output through `co::dtoa`, with at most 16 significant decimal places by default.
- Pointers are output as `0x` + hexadecimal.
- `nullptr` outputs `0x0`.
- `decimal` allows specifying the number of significant decimal places.

`decimal` definition:

```cpp
struct decimal {
    constexpr decimal(double v, int n) noexcept : v(v), n(n) {}
    double v;
    int n; // significant decimal places
};
```

Example:

```cpp
co::string s;
s << "hello " << 23 << ' ' << 3.14 << ' ' << true;
s << co::decimal(3.14159, 2); // keep two significant decimal places
```

### cat

```cpp
string& cat() noexcept;
template<typename X, typename ...V>
string& cat(X&& x, V&& ... v) noexcept;
```

- `cat(...)` applies `<<` to the current string for each argument in turn.
- It is equivalent to chained `<<`, but more concise.

Example:

```cpp
co::string s;
s.cat("hello ", 23, ' ', 3.14);
```

### Comparison and Search

#### compare

```cpp
int compare(const char* s, size_t n) const noexcept;
int compare(const char* s) const noexcept;
int compare(const string& s) const noexcept;
int compare(const std::string& s) const noexcept;
```

Returns:

- `< 0`: current string is less than `s`;
- `0`: equal;
- `> 0`: current string is greater than `s`.

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

#### ifind (case-insensitive)

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

All searches return `co::string::npos` on failure.

### Match, Case, Substring

```cpp
// * matches any character, ? matches a single character
bool match(const char* pattern) const noexcept;

// Convert string to lowercase/uppercase
string& tolower() noexcept;
string& toupper() noexcept;

// Return a lowercase/uppercase copy of the string
string lower() const noexcept;
string upper() const noexcept;

string substr(size_t pos) const noexcept;
string substr(size_t pos, size_t len) const noexcept;
```

### Escape and Unescape

```cpp
// Escape characters: '"'、'\\'、'\0'、'\r'、'\n'、'\t'、'\a'、'\b'、'\f'、'\v'
string& escape() noexcept;
string& unescape() noexcept;
```

### Remove, Trim, Replace

```cpp
string& remove_outer(size_t n) noexcept;        // remove n characters from both ends
string& remove_prefix(size_t n) noexcept;       // remove n characters from the prefix
string& remove_suffix(size_t n) noexcept;       // remove n characters from the suffix

// Remove prefix s
string& remove_prefix(const char* s, size_t n) noexcept;
string& remove_prefix(const char* s) noexcept;
string& remove_prefix(const string& s) noexcept;
string& remove_prefix(const std::string& s) noexcept;

// Remove suffix s
string& remove_suffix(const char* s, size_t n) noexcept;
string& remove_suffix(const char* s) noexcept;
string& remove_suffix(const string& s) noexcept;
string& remove_suffix(const std::string& s) noexcept;

// Remove character c from both sides of the string
string& trim(char c) noexcept;
string& trim_left(char c) noexcept;
string& trim_right(char c) noexcept;

// Remove characters in s from both sides of the string
string& trim(const char* s=" \t\r\n") noexcept;
string& trim_left(const char* s=" \t\r\n") noexcept;
string& trim_right(const char* s=" \t\r\n") noexcept;

// @t: maximum number of replacements, 0 means no limit
string& replace(const char* sub, size_t n, const char* to, size_t m, size_t t=0) noexcept;
string& replace(const char* sub, const char* to, size_t t=0) noexcept;
string& replace(const string& sub, const string& to, size_t t=0) noexcept;
```

## Numeric Conversion Functions

### itoa / utoh / ptoh / dtoa

```cpp
// Integer to decimal string
template<typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
inline int itoa(T v, char* buf, uint32 buf_size);

// Unsigned integer to hexadecimal (with 0x prefix)
template<typename T, typename = std::enable_if_t<std::is_integral_v<T> && std::is_unsigned_v<T>>>
inline int utoh(T v, char* buf, uint32 buf_size);

// Pointer to hexadecimal (with 0x prefix)
inline int ptoh(const void* v, char* buf, uint32 buf_size);

// double to string, buf length must be at least 25
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

### String to Number

```cpp
int32  stoi32(const char* s, int* err=nullptr) noexcept;
int64  stoi64(const char* s, int* err=nullptr) noexcept;
uint32 stou32(const char* s, int* err=nullptr) noexcept;
uint64 stou64(const char* s, int* err=nullptr) noexcept;
int    stoi  (const char* s, int* err=nullptr) noexcept;
bool   stob  (const char* s, int* err=nullptr) noexcept;
double stod  (const char* s, int* err=nullptr) noexcept;
```

- Overloads for both `co::string` and `std::string` are provided.
- On failure, the error code is returned through `err`; possible error codes include `ERANGE` and `EINVAL`.

## Memory Helper Functions

```cpp
char* co::memrchr(const char* s, char c, size_t n);

// Return the position of the first occurrence of p in s
char* co::memmem (const char* s, size_t n, const char* p, size_t m);

// Case-insensitive, return the position of the first occurrence of p in s
char* co::memimem(const char* s, size_t n, const char* p, size_t m);

// Return the position of the last occurrence of p in s
char* co::memrmem(const char* s, size_t n, const char* p, size_t m);

// Compare two memory segments of lengths n and m
// If lengths differ, return -1 or 1
inline int co::memcmp(const char* s, size_t n, const char* p, size_t m);
```

## Global Convenience Functions

### split

```cpp
vector<string> split(const char* s, size_t n, char c, size_t t=0);
vector<string> split(const char* s, size_t n, const char* c, size_t m, size_t t=0);

inline vector<string> split(const char* s, char c, size_t t=0);
inline vector<string> split(const string& s, char c, size_t t=0);
inline vector<string> split(const char* s, const char* c, size_t t=0);
inline vector<string> split(const string& s, const char* c, size_t t=0);
```

- `t` is the maximum number of splits; `t = 0` means no limit.

Example:

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

## operator+ and Comparison Operators

```cpp
inline co::string operator+(const co::string& a, char b) noexcept;
inline co::string operator+(char a, const co::string& b) noexcept;
inline co::string operator+(const co::string& a, const co::string& b) noexcept;
inline co::string operator+(const co::string& a, const std::string& b) noexcept;
inline co::string operator+(const std::string& a, const co::string& b) noexcept;
inline co::string operator+(const co::string& a, const char* b) noexcept;
inline co::string operator+(const char* a, const co::string& b) noexcept;
```

Comparison operators support:

- `co::string` with `co::string`
- `co::string` with `std::string`
- `co::string` with `const char*`

Supports `==`, `!=`, `<`, `>`, `<=`, `>=`; only contents are compared, regardless of the underlying implementation.

## std::hash Specialization

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

- `co::string` can be used directly in `std::unordered_map` / `std::unordered_set`.
- The hash is based on `co::murmur_hash`.

## co::vector Alias

```cpp
template<class T, class Alloc = co::stl_allocator<T>>
using vector = std::vector<T, Alloc>;
```

- `co::vector<T>` is `std::vector` using the coost allocator.
- `co::split` returns `co::vector<co::string>`.

## Interoperability with std::ostream

```cpp
inline std::ostream& operator<<(std::ostream& os, const co::string& s) noexcept {
    return os.write(s.data(), s.size());
}
```

## Example

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
