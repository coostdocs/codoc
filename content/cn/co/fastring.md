---
weight: 2
title: "字符串(fastring)"
---

include: [co/fastring.h](https://github.com/idealvin/co/blob/master/include/co/fastring.h).


## fastring

`fastring` 是 co 中的字符串类型，它实现了 std::string 中的大部分方法，同时也提供了一些 std::string 没有的方法。



### fastring::fastring

```cpp
1.  fastring() noexcept;
2.  explicit fastring(size_t cap);
3.  fastring(const void* s, size_t n);
4.  fastring(const char* s);
5.  fastring(const std::string& s);
6.  fastring(size_t n, char c);
7.  fastring(char c, size_t n);
8.  fastring(const fastring& s);
9.  fastring(fastring&& s) noexcept;
```

- 第 1 个是默认构造函数，创建一个空的 fastring 对象，不会分配任何内存。
- 第 2 个构造函数同样创建一个空的 fastring 对象，但用参数 cap 指定初始容量，即预分配 cap 字节的内存。
- 第 3 个构造函数从给定的字节序列创建 fastring 对象，参数 n 是序列长度。
- 第 4 个构造函数从 C 风格的字符串创建 fastring 对象，s 必须是 '\0' 结尾的字符串。
- 第 5 个构造函数从 std::string 创建一个 fastring 对象。
- 第 6 个与第 7 个构造函数，将 fastring 对象初始化为 n 个字符 c 构成的字符串。
- 第 8 个是拷贝构造函数，内部会进行内存拷贝。
- 第 9 个是 move 构造函数，不会进行内存拷贝。


- 示例

```cpp
fastring s;                // 空字符串，无内存分配
fastring s(32);            // 空字符串，预分配内存(容量为32)
fastring s("hello");       // 初始化 s 为 "hello"
fastring s("hello", 3);    // 初始化 s 为 "hel"
fastring s(88, 'x');       // 初始化 s 为 88 个 'x'
fastring s('x', 88);       // 初始化 s 为 88 个 'x'
fastring t(s);             // 拷贝构造
fastring x(std::move(s));  // 移动构造，s 自身变成空字符串
```



### fastring::operator=

```cpp
fastring& operator=(const char* s);
fastring& operator=(const std::string& s);
fastring& operator=(const fastring& s);
fastring& operator=(fastring&& s) noexcept;
```

- 第 1 个用 '\0' 结尾的字符串进行赋值，s 可以是进行赋值操作的 fastring 的一部分。
- 第 2 个用 std::string 进行赋值。
- 第 3 个是拷贝赋值操作，若 s 与进行赋值操作的 fastring 是同一个对象，则不会进行任何操作。
- 第 4 个是 move 赋值操作，s 自身会变成空字符串。

- 示例

```cpp
fastring s;
fastring t;
s = "hello";
s = s.c_str() + 2;     // s -> "llo"   
s = std::string("x");
t = s;
t = std::move(s);
```



### ———————————
### fastring::back

```cpp
char& back() const;
```

- 此方法返回 fastring 中最后一个字符的引用。
- 若 fastring 为空，调用此方法会导致未定义的行为。

- 示例

```cpp
fastring s("hello");
char c = s.back();   // c = 'o'
s.back() = 'x';      // s -> "hellx"
```



### fastring::front

```cpp
char& front() const;
```

- 此方法返回 fastring 中第一个字符的引用。
- 若 fastring 为空，调用此方法会导致未定义的行为。

- 示例

```cpp
fastring s("hello");
char c = s.front();  // c = 'h'
s.front() = 'x';     // s -> "xello"
```



### fastring::operator[]

```cpp
char& operator[](size_t n) const;
```

- 此方法返回 fastring 中第 n 个字符的引用。
- 若 n 超出合理的范围，调用此方法会导致未定义的行为。

- 示例

```cpp
fastring s("hello");
char c = s[1];       // c = 'e'
s[1] = 'x';          // s -> "hxllo"
```



### ———————————
### fastring::capacity

```cpp
size_t capacity() const;
```

- 此方法返回 fastring 的容量。



### fastring::c_str

```cpp
const char* c_str() const;
```

- 此方法获取等效的 C 字符串。
- 此方法在 fastring 末尾加上一个 '\0'，它不会改变 fastring 的 size 及内容，但有可能导致内部重新分配内存。



### fastring::data

```cpp
const char* data() const;
```

- 此方法与 c_str() 类似，但不保证字符串以 '\0' 结尾。



### fastring::empty

```cpp
bool empty() const;
```

- 此方法判断 fastring 是否为空。



### fastring::size

```cpp
size_t size() const;
```

- 此方法返回 fastring 的长度。



### fastring::substr

```cpp
fastring substr(size_t pos) const;
fastring substr(size_t pos, size_t len) const;
```

- 第 1 个版本返回从位置 pos 开始的子串。
- 第 2 个版本返回从位置 pos 开始、长度为 len 的子串。

- 示例

```cpp
fastring s("hello");
s.substr(2);     // "llo"
s.substr(2, 2);  // "ll"
```



### ———————————
### fastring::append

```cpp
1.  fastring& append(const void* s, size_t n);
2.  fastring& append(const char* s);
3.  fastring& append(const fastring& s);
4.  fastring& append(const std::string& s);
5.  fastring& append(size_t n, char c);
6.  fastring& append(char c, size_t n);
7.  fastring& append(char c);
8.  fastring& append(signed char c);
9.  fastring& append(unsigned char c);
```

- 第 1 个版本追加指定长度的字节序列，n 为序列长度。
- 第 2 个版本追加 '\0' 结尾的字符串，s 可以是执行 append 操作的 fastring 的一部分。
- 第 3 个版本追加 fastring 对象，s 可以是执行 append 操作的 fastring 对象本身。
- 第 4 个版本追加 std::string 对象。
- 第 5 个与第 6 个版本追加 n 个字符 c。
- 第 7 到 9 个版本追加单个字符 c。
- 此方法返回 fastring 的引用，多个 append 操作可以连写到一行。


- 示例

```cpp
fastring s;
s.append('c');                 // s -> "c"
s.append(2, 'c');              // s -> "ccc"
s.append('c', 2);              // s -> "ccccc"
s.clear();
s.append('c').append(2, 'x');  // s -> "cxx"
s.append(s.c_str() + 1);       // s -> "cxxxx"
s.append(s.data(), 3);         // s -> "cxxxxcxx"
```



### fastring::cat

```cpp
template<typename X, typename ...V>
fastring& cat(X&& x, V&& ... v);
```

- v2.0.3 新增。将任意数量的元素连接到 fastring 中。
- 此方法调用 `operator<<` 操作，将参数中的元素逐个追加到 fastring 中。

- 示例

```cpp
fastring s("hello");
s.cat(' ', 23, "xx", false); // s -> "hello 23xxfalse"
```



### fastring::operator<<

```cpp
fastring& operator<<(const signed char* s);
fastring& operator<<(const unsigned char* s);
template<typename T> fastring& operator<<(T&& t);
```

- 第 1, 2 个版本 v2.0.3 中新增，等价于 `fastring& operator<<(const char* s)`。
- 第 3 个版本中，T 可以是任意的基本类型(bool, char, int, double, void* 等)，以及字符串类型(const char*, fastring, std::string)。
- 字符串类型的参数，可以是执行 `operator<<` 操作的 fastring 本身或它的一部分。

- 示例

```cpp
fastring s;
s << false;           // s -> "false"
s << s;               // s -> "falsefalse"    (append itself)
s.clear();
s << "hello " << 23;  // s -> "hello 23"
s << s.c_str() + 6;   // s -> "hello 2323"    (append part of s)
s << ' ';             // s -> "hello 2323 "
```



### fastring::operator+=

```cpp
fastring& operator+=(const char* s);
fastring& operator+=(const fastring& s);
fastring& operator+=(const std::string& s);
fastring& operator+=(char c);
fastring& operator+=(signed char c);
fastring& operator+=(unsigned char c);
```

- 此方法等价于 [append()](#fastringappend) 方法。

- 示例

```cpp
fastring s;
s += 'c';   // s -> "c"
s += "xx";  // s -> "cxx"
```



### ———————————
### fastring::find

```cpp
1. size_t find(char c) const;
2. size_t find(char c, size_t pos) const;
3. size_t find(char c, size_t pos, size_t len) const;
4. size_t find(const char* s) const;
5. size_t find(const char* s, size_t pos) const;
```

- 1, 从位置 0 开始查找字符 c。
- 2, 从位置 pos 开始查找字符 c。
- 3, v3.0 新增，在 `[pos, pos + len)` 范围内查找字符 c。
- 4, 从位置 0 开始查找子串 s，内部基于 `strstr()` 实现，不适用于包含 '\0' 的 fastring。
- 5, 从位置 pos 开始查找子串 s，内部基于 `strstr()` 实现，不适用于包含 '\0' 的 fastring。
- 此方法查找成功时，返回所查找字符或子串的位置，否则返回 `fastring::npos`。


- 示例

```cpp
fastring s("hello");
s.find('l');      // return 2
s.find('l', 3);   // return 3
s.find("ll");     // return 2
s.find("ll", 3);  // return s.npos
```



### fastring::rfind

```cpp
size_t rfind(char c) const;
size_t rfind(const char* s) const;
```

- 第 1 个版本反向查找单个字符，基于 `strrchr()` 实现，不适用于包含 '\0' 的 fastring。
- 第 2 个版本反向查找子串。
- 此方法查找成功时，返回所查找字符或子串的位置，否则返回 fastring::npos。

- 示例

```cpp
fastring s("hello");
s.rfind('l');   // return 3
s.rfind("ll");  // return 2
s.rfind("le");  // return s.npos
```



### fastring::find_first_of

```cpp
size_t find_first_of(const char* s) const;
size_t find_first_of(const char* s, size_t pos) const;
```

- 查找第一个出现的指定字符集中的字符。
- 第 1 个版本从位置 0 开始查找。
- 第 2 个版本从位置 pos 开始查找。
- 此方法基于 `strcspn()` 实现，不适用于包含 '\0' 的 fastring。
- 此方法从头开始查找，遇到 s 中的任意字符时，即返回该字符的位置，否则返回 `fastring::npos`。

- 示例

```cpp
fastring s("hello");
s.find_first_of("def");    // return 1
s.find_first_of("ol", 3);  // return 3
```



### fastring::find_first_not_of

```cpp
size_t find_first_not_of(const char* s) const;
size_t find_first_not_of(const char* s, size_t pos) const;
size_t find_first_not_of(char s, size_t pos=0);
```

- 查找第一个出现的非指定字符集中的字符。
- 第 1 个版本从位置 0 开始查找。
- 第 2 个版本从位置 pos 开始查找。
- 第 3 个版本字符集为单个字符，s 不能是 '\0'。
- 此方法基于 `strspn` 实现，不适用于包含 '\0' 的 fastring。
- 此方法从头开始查找，遇到非 s 中的任意字符时，即返回该字符的位置，否则返回 `fastring::npos`。

- 示例

```cpp
fastring s("hello");
s.find_first_not_of("he");     // return 2
s.find_first_not_of("he", 3);  // return 3
s.find_first_not_of('x');      // return 0
```



### fastring::find_last_of

```cpp
size_t find_last_of(const char* s, size_t pos=npos) const;
```

- 查找最后一个出现的指定字符集中的字符。
- 此方法中参数 pos 默认为 npos，即从字符串尾部开始查找。
- 此方法从 pos 处开始反向查找，遇到 s 中的任意字符时，即返回该字符的位置，否则返回 `fastring::npos`。

- 示例

```cpp
fastring s("hello");
s.find_last_of("le");     // return 3
s.find_last_of("le", 1);  // return 1
```



### fastring::find_last_not_of

```cpp
size_t find_last_not_of(const char* s, size_t pos=npos) const;
size_t find_last_not_of(char s, size_t pos=npos) const;
```

- 查找最后一个出现的非指定字符集中的字符。
- 此方法中参数 pos 默认为 npos，即从字符串尾部开始查找。
- 第 2 个版本中 s 是单个字符，s 不能是 '\0'。
- 此方法从 pos 处开始反向查找，遇到非 s 中的任意字符时，即返回该字符的位置，否则返回 `fastring::npos`。

- 示例

```cpp
fastring s("hello");
s.find_last_not_of("le");     // return 4
s.find_last_not_of("le", 3);  // return 0
s.find_last_not_of('o');      // return 3
```



### fastring::npos

```cpp
static const size_t npos = (size_t)-1;
```

- npos 是 size_t 类型的最大值。
- npos 作为长度时，表示直到字符串结尾。
- npos 作为返回值时，表示未找到。

- 示例

```cpp
fastring s("hello");
size_t r = s.find('x');
r == s.npos;  // true
```



### ———————————
### fastring::replace

```cpp
fastring& replace(const char* sub, const char* to, size_t maxreplace=0);
```

- 此方法将 fastring 中的子串 sub 替换成 to，参数 maxreplace 指定最大的替换次数，0 表示不限次数。
- 此方法返回 fastring 的引用，多个 replace 操作可以连写到一行。

- 示例

```cpp
fastring s("hello");
s.replace("ll", "rr");                     // s -> "herro"
s.replace("err", "ok").replace("k", "o");  // s -> "hooo"
```



### fastring::strip

```cpp
fastring& strip(const char* s=" \t\r\n", char d='b');
fastring& strip(char s, char d='b');
```

- 修剪字符串，去掉 fastring 左边、右边或两边的指定字符。
- 参数 s 为要修剪的字符，参数 d 表示方向，'l' 或 'L' 表示左边，'r' 或 'R' 表示右边，默认为 'b' 表示左右两边。
- 第 1 个版本默认去掉字符串两边的空白字符。
- 第 2 个版本中 s 为单个字符，s 不能是 '\0'。

- 示例

```cpp
fastring s = " sos\r\n";
s.strip();          // s -> "sos"
s.strip('s', 'l');  // s -> "os"
s.strip('s', 'r');  // s -> "o"
```



### fastring::tolower

```cpp
fastring& tolower();
```

- 此方法将 fastring 转换成小写，并返回 fastring 的引用。



### fastring::toupper

```cpp
fastring& toupper();
```

- 此方法将 fastring 转换成大写，并返回 fastring 的引用。



### fastring::lower

```cpp
fastring lower() const;
```

- 此方法返回 fastring 的小写形式。



### fastring::upper

```cpp
fastring upper() const;
```

- 此方法返回 fastring 的大写形式。


- 示例

```cpp
fastring s("Hello");
fastring x = s.lower();  // x = "hello", s 保持不变
fastring y = s.upper();  // x = "HELLO", s 保持不变
s.tolower();             // s -> "hello"
s.toupper();             // s -> "HELLO"
```



### ———————————
### fastring::clear

```cpp
void clear();
```

- 此方法仅将 fastring 的 size 置为 0，capacity 保持不变。



### fastring::ensure

```cpp
void ensure(size_t n);
```

- 此方法确保 fastring 剩余的内存能容纳至少 n 个字符。



### fastring::reserve

```cpp
void reserve(size_t n);
```

- 此方法调整 fastring 的容量，确保容量至少是 n。
- 当 n 小于原来的容量时，则保持容量不变。



### fastring::resize

```cpp
void resize(size_t n);
```

- 此方法将 fastring 的 size 设置为 n。
- 当 n 大于原来的 size 时，此操作将 size 扩大到 n，但不会用 '\0' 填充扩展的部分。

- 示例

```cpp
fastring s("hello");
s.resize(3);    // s -> "hel"
s.resize(6);
char c = s[5];  // c 是不确定的随机值
```



### fastring::reset

```cpp
void reset();
```

- v2.0.3 新增。清空 fastring 并释放内存。



### fastring::safe_clear

```cpp
void safe_clear();
```

- 与 `clear()` 类似，但会对内部内存清零。



### fastring::swap

```cpp
void swap(fastring& s) noexcept;
void swap(fastring&& s) noexcept;
```

- 此方法交换两个 fastring，仅交换内部指针、容量、大小。

- 示例

```cpp
fastring s("hello");
fastring x("world");
s.swap(x);  // s -> "world", x -> "hello"
```



### ———————————
### fastring::starts_with

```cpp
bool starts_with(char s) const;
bool starts_with(const char* s, size_t n) const;
bool starts_with(const char* s) const;
bool starts_with(const fastring& s) const;
bool starts_with(const std::string& s) const;
```

- 此方法判断 fastring 是否以 s 开头，s 是单个字符或字符串。
- 当 s 为空字符串时，此方法始终返回 true。



### fastring::ends_with

```cpp
bool ends_with(char s) const;
bool ends_with(const char* s, size_t n) const;
bool ends_with(const char* s) const;
bool ends_with(const fastring& s) const;
bool ends_with(const std::string& s) const;
```

- 此方法判断 fastring 是否以 s 结尾，s 是单个字符或字符串。
- 当 s 为空字符串时，此方法始终返回 true。



### fastring::match

```cpp
bool match(const char* pattern) const;
```

- 判断 fastring 是否匹配模式 pattern，`*` 匹配任意字符串，`?` 匹配单个字符。

- 示例

```cpp
fastring s("hello");
s.match("he??o");  // true
s.match("h*o");    // true
s.match("he?o");   // false
s.match("*o");     // true
s.match("h*l?");   // true
```



### fastring::lshift

```cpp
fastring& lshift(size_t n);
```

- 此方法将 fastring 左移 n 个字符，也就是删除前 n 个字符。

- 示例

```cpp
fastring s("hello");
s.lshift(2);          // s -> "llo"
s.lshift(8);          // s -> ""
```



### fastring::remove_tail

```cpp
fastring& remove_tail(const char* s, size_t n);
fastring& remove_tail(const char* s);
fastring& remove_tail(const fastring& s);
fastring& remove_tail(const std::string& s);
```

- 此方法删除 fastring 尾部的字符串 s，仅当 fastring 以 s 结尾时，才会删除。

- 示例

```cpp
fastring s("hello.log");
s.remove_tail(".log");  // s -> "hello"
```



### fastring::shrink

```cpp
void shrink();
```

- 此方法释放 fastring 中多余的内存。

- 示例

```cpp
fastring s("hello");
s.reserve(32);  // capacity -> 32
s.shrink();     // capacity -> 6
```




## global functions


### operator+

```cpp
fastring operator+(const fastring& a, char b);
fastring operator+(char a, const fastring& b);
fastring operator+(const fastring& a, const fastring& b);
fastring operator+(const fastring& a, const char* b);
fastring operator+(const char* a, const fastring& b);
fastring operator+(const fastring& a, const std::string& b);
fastring operator+(const std::string& a, const fastring& b);
```

- 加法操作，此方法至少有一个参数是 fastring。

- 示例

```cpp
fastring s;
s = s + '^';        // s -> "^"
s = "o" + s + "o";  // s -> "o^o"
```



### operator==

```cpp
bool operator==(const fastring& a, const fastring& b);
bool operator==(const fastring& a, const char* b);
bool operator==(const char* a, const fastring& b);
bool operator==(const fastring& a, const std::string& b);
bool operator==(const std::string& a, const fastring& b);
```

- 此方法判断两个字符串是否相等，至少有一个参数是 fastring。



### operator!=

```cpp
bool operator!=(const fastring& a, const fastring& b);
bool operator!=(const fastring& a, const char* b);
bool operator!=(const char* a, const fastring& b);
bool operator!=(const fastring& a, const std::string& b);
bool operator!=(const std::string& a, const fastring& b);
```

- 此方法判断两个字符串是否不相等，至少有一个参数是 fastring。



### operator<

```cpp
bool operator<(const fastring& a, const fastring& b);
bool operator<(const fastring& a, const char* b);
bool operator<(const char* a, const fastring& b);
bool operator<(const fastring& a, const std::string& b);
bool operator<(const std::string& a, const fastring& b);
```

- 此方法判断字符串 a 是否小于 b，至少有一个参数是 fastring。



### operator>

```cpp
bool operator>(const fastring& a, const fastring& b);
bool operator>(const fastring& a, const char* b);
bool operator>(const char* a, const fastring& b);
bool operator>(const fastring& a, const std::string& b);
bool operator>(const std::string& a, const fastring& b);
```

- 此方法判断字符串 a 是否大于 b，至少有一个参数是 fastring。



### operator<=

```cpp
bool operator<=(const fastring& a, const fastring& b);
bool operator<=(const fastring& a, const char* b);
bool operator<=(const char* a, const fastring& b);
bool operator<=(const fastring& a, const std::string& b);
bool operator<=(const std::string& a, const fastring& b);
```

- 此方法判断字符串 a 是否小于或等于 b，至少有一个参数是 fastring。



### operator>=

```cpp
bool operator>=(const fastring& a, const fastring& b);
bool operator>=(const fastring& a, const char* b);
bool operator>=(const char* a, const fastring& b);
bool operator>=(const fastring& a, const std::string& b);
bool operator>=(const std::string& a, const fastring& b);
```

- 此方法判断字符串 a 是否大于或等于 b，至少有一个参数是 fastring。


- 示例

```cpp
fastring s("hello");
s == "hello";  // true
s != "hello";  // false
s > "aa";      // true
s < "xx";      // true
s >= "he";     // true
s <= "he";     // false
```



### operator<<

```cpp
std::ostream& operator<<(std::ostream& os, const fastring& s);
```

- 示例

```cpp
fastring s("xx");
std::cout << s << std::endl;
```

