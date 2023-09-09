---
weight: 3
title: "fastring"
---

include: [co/fastring.h](https://github.com/idealvin/coost/blob/master/include/co/fastring.h).


## fastring

**fastring** 是 coost 提供的字符串类型，它实现了 std::string 中的大部分方法，同时也提供了一些 std::string 没有的方法。


### constructor

```cpp
1.  constexpr fastring() noexcept;
2.  explicit fastring(size_t cap);
3.  fastring(const void* s, size_t n);
4.  fastring(const char* s);
5.  fastring(const std::string& s);
6.  fastring(size_t n, char c);
7.  fastring(char c, size_t n);
8.  fastring(const fastring& s);
9.  fastring(fastring&& s) noexcept;
```

- 1, 默认构造函数，创建一个空的 fastring 对象，不会分配任何内存。
- 2, 创建一个空的 fastring 对象，但用参数 cap 指定初始容量，即预分配 cap 字节的内存。
- 3, 用给定的字节序列创建 fastring 对象，参数 n 是序列长度。
- 4, 用 C 风格的字符串创建 fastring 对象，s 必须是 '\0' 结尾的字符串。
- 5, 用 std::string 创建一个 fastring 对象。
- 6-7, 将 fastring 对象初始化为 n 个字符 c 构成的字符串。
- 8, 拷贝构造函数，内部会进行内存拷贝。
- 9, 移动构造函数，不会进行内存拷贝。


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



### operator=

```cpp
1. fastring& operator=(const char* s);
2. fastring& operator=(const std::string& s);
3. fastring& operator=(const fastring& s);
4. fastring& operator=(fastring&& s) noexcept;
```

- 1, 用 '\0' 结尾的字符串进行赋值，s 可以是进行赋值操作的 fastring 的一部分。
- 2, 用 std::string 进行赋值。
- 3, 拷贝赋值操作。
- 4, 移动赋值操作，s 自身会变成空字符串。

- 示例

```cpp
fastring s;
fastring t;
s = "hello";           // s -> "hello"
s = s;                 // nothing will be done
s = s.c_str() + 2;     // s -> "llo"
s = std::string("x");  // s -> "x"
t = s;                 // t -> "x"
t = std::move(s);      // t -> "x", s -> ""
```



### assign

```cpp
1. fastring& assign(const void* s, size_t n);
2. template<typename S> fastring& assign(S&& s);
```

- 对 fastring 进行赋值，v3.0.1 新增。
- 1 中 n 是字节序列的长度，2 与 [operator=](#operator) 等价。



### ———————————
### back

```cpp
char& back();
const char& back() const;
```

- 此方法返回 fastring 中最后一个字符的引用。

{{< hint warning >}}
若 fastring 为空，调用此方法会导致未定义的行为。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
char c = s.back();   // c = 'o'
s.back() = 'x';      // s -> "hellx"
```



### front

```cpp
char& front();
const char& front() const;
```

- 此方法返回 fastring 中第一个字符的引用。

{{< hint warning >}}
若 fastring 为空，调用此方法会导致未定义的行为。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
char c = s.front();  // c = 'h'
s.front() = 'x';     // s -> "xello"
```



### operator[]

```cpp
char& operator[](size_t n);
const char& operator[](size_t n) const;
```

- 此方法返回 fastring 中第 n 个字符的引用。

{{< hint warning >}}
若 n 超出合理的范围，调用此方法会导致未定义的行为。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
char c = s[1];       // c = 'e'
s[1] = 'x';          // s -> "hxllo"
```



### ———————————
### capacity

```cpp
size_t capacity() const noexcept;
```

- 此方法返回 fastring 的容量。



### c_str

```cpp
const char* c_str() const;
```

- 此方法获取等效的 C 字符串。
- 此方法在 fastring 末尾加上一个 '\0'，它不会改变 fastring 的 size 及内容，但有可能导致内部重新分配内存。



### data

```cpp
const char* data() const noexcept;
```

- 此方法与 [c_str()](#c_str) 类似，但不保证字符串以 '\0' 结尾。



### empty

```cpp
bool empty() const noexcept;
```

- 此方法判断 fastring 是否为空。



### size

```cpp
size_t size() const noexcept;
```

- 此方法返回 fastring 的长度。



### ———————————
### clear

```cpp
1. void clear();
2. void clear(char c);
```

- 此方法仅将 fastring 的 size 置为 0，capacity 保持不变。
- 2 与 1 类似，只是 size 置为 0 前会用字符 c 填充内部内存。

{{< hint info >}}
2 是 v3.0.1 新增，可用于清除内存中的敏感信息。
{{< /hint >}}


### ensure

```cpp
void ensure(size_t n);
```

- 此方法确保 fastring 剩余的内存能容纳至少 n 个字符。



### reserve

```cpp
void reserve(size_t n);
```

- 此方法调整 fastring 的容量，确保容量至少是 n。
- 当 n 小于原来的容量时，则保持容量不变。



### reset

```cpp
void reset();
```

- v2.0.3 新增。清空 fastring 并释放内存。



### resize

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



### shrink

```cpp
void shrink();
```

- 释放 fastring 中多余的内存。

- 示例

```cpp
fastring s("hello");
s.reserve(32);  // capacity -> 32
s.shrink();     // capacity -> 6
```



### swap

```cpp
void swap(fastring& s) noexcept;
void swap(fastring&& s) noexcept;
```

- 交换两个 fastring，仅交换内部指针、容量、大小。

- 示例

```cpp
fastring s("hello");
fastring x("world");
s.swap(x);  // s -> "world", x -> "hello"
```




### ———————————
### append

```cpp
1. fastring& append(const void* s, size_t n);
2. fastring& append(const char* s);
3. fastring& append(const fastring& s);
4. fastring& append(const std::string& s);
5. fastring& append(size_t n, char c);
6. fastring& append(char c, size_t n);
7. fastring& append(char c);
```

- 1, 追加指定长度的字节序列，n 为序列长度。
- 2, 追加 '\0' 结尾的字符串，s 可以是执行 append 操作的 fastring 的一部分。
- 3, 追加 fastring 对象，s 可以是执行 append 操作的 fastring 对象本身。
- 4, 追加 std::string 对象。
- 5-6, 追加 n 个字符 c。
- 7, 追加单个字符 c。

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


### append_nomchk

```cpp
fastring& append_nomchk(const void* s, size_t n);
fastring& append_nomchk(const char* s)
```

- 与 [append()](#append) 类似，但不会检查 s 是否与内部内存重叠。

{{< hint warning >}}
若 s 与 fastring 内部内存可能重叠，则不能使用此方法。
{{< /hint >}}




### cat

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



### operator<<

```cpp
fastring& operator<<(bool v);

fastring& operator<<(char v);
fastring& operator<<(signed char v);
fastring& operator<<(unsigned char v);

fastring& operator<<(short v);
fastring& operator<<(unsigned short v);
fastring& operator<<(int v);
fastring& operator<<(unsigned int v);
fastring& operator<<(long v);
fastring& operator<<(unsigned long v);
fastring& operator<<(long long v);
fastring& operator<<(unsigned long long v);

fastring& operator<<(double v);
fastring& operator<<(float v);
fastring& operator<<(const dp::_fpt& v);

fastring& operator<<(const void* v);
fastring& operator<<(std::nullptr_t);

fastring& operator<<(const char* s);
fastring& operator<<(const signed char* s);
fastring& operator<<(const unsigned char* s);
fastring& operator<<(const fastring& s);
fastring& operator<<(const std::string& s);
```

- 将 bool、char、整数类型、浮点数类型、指针类型、字符串类型的值格式化后追加到 fastring 中。
- `operator<<(const dp::_fpt&)` 用于格式化输出浮点数，可指定有效小数位数。

- 示例

```cpp
fastring s;
s << 'x';             // s -> "x"
s << s;               // s -> "xx"           (append itself)
s << false;           // s -> "xxfalse"

s.clear();
s << "hello " << 23;  // s -> "hello 23"
s << (s.c_str() + 6); // s -> "hello 2323"   (append part of s)

s.clear();
s << 3.1415;          // s -> "3.1415"

s.clear();
s << (void*)32;       // s -> "0x20"
```

{{< hint info >}}
**指定有效小数的位数**  
coost 提供 `dp::_1, dp::_2, ..., dp::_16, dp::_n`，用于设置浮点数的有效小数位数。
```cpp
fastring s;
s << dp::_2(3.1415);   // "3.14
s << dp::_3(3.1415);   // "3.141"
s << dp::_n(3.14, 1);  // "3.1", 与 dp::_1(3.14) 等价
```
{{< /hint >}}




### operator+=

```cpp
fastring& operator+=(const char* s);
fastring& operator+=(const fastring& s);
fastring& operator+=(const std::string& s);
fastring& operator+=(char c);
```

- 此方法等价于 [append()](#append) 方法。

- 示例

```cpp
fastring s;
s += 'c';   // s -> "c"
s += "xx";  // s -> "cxx"
```



### push_back

```cpp
fastring& push_back(char c);
```

- 追加单个字符到 fastring 尾部，与 [append(c)](#append) 等价。



### pop_back

```cpp
char pop_back();
```

- 取出并返回 fastring 最后的一个字符。

{{< hint warning >}}
若 fastring 为空，调用此方法会导致未定义的行为。
{{< /hint >}}



### ———————————
### compare

```cpp
 1. int compare(const char* s, size_t n) const;
 2. int compare(const char* s) const;
 3. int compare(const fastring& s) const noexcept;
 4. int compare(const std::string& s) const noexcept;

 5. int compare(size_t pos, size_t len, const char* s, size_t n) const;
 6. int compare(size_t pos, size_t len, const char* s) const;
 7. int compare(size_t pos, size_t len, const fastring& s) const;
 8. int compare(size_t pos, size_t len, const std::string& s) const;

 9. int compare(size_t pos, size_t len, const fastring& s, size_t spos, size_t n=npos) const;
10. int compare(size_t pos, size_t len, const std::string& s, size_t spos, size_t n=npos) const;
```

- 比较 fastring 与指定的字符串。
- 1-4, 将 fastring 与 s 比较，n 是 s 的长度。
- 5-8, 将 fastring 的 `[pos, pos+len)` 部分与 s 进行比较，n 是 s 的长度。
- 9-10, 将 fastring 的 `[pos, pos+len)` 部分与 s 的 `[spos, spos+n)` 部分进行比较。



### contains

```cpp
bool contains(char c) const;
bool contains(const char* s) const;
bool contains(const fastring& s) const;
bool contains(const std::string& s) const;
```

- 判断 fastring 中是否包含指定的字符或字符串。



### find

```cpp
1. size_t find(char c) const;
2. size_t find(char c, size_t pos) const;
3. size_t find(char c, size_t pos, size_t len) const;
4. size_t find(const char* s) const;
5. size_t find(const char* s, size_t pos) const;
6. size_t find(const char* s, size_t pos, size_t n) const;
7. size_t find(const fastring& s, size_t pos=0) const;
8. size_t find(const std::string& s, size_t pos=0) const;
```

- 1, 从位置 0 开始查找字符 c。
- 2, 从位置 pos 开始查找字符 c。
- 3, v3.0 新增，在 `[pos, pos + len)` 范围内查找字符 c。
- 4, 从位置 0 开始查找子串 s。
- 5, 从位置 pos 开始查找子串 s。
- 6, 从位置 pos 开始查找子串 s，s 长度为 n。
- 7-8, 从位置 pos 开始查找子串 s。
- 此方法查找成功时，返回所查找字符或子串的位置，否则返回 `npos`。

{{< hint info >}}
v3.0.1 新增上述 6-8，并支持二进制字符串的查找。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
s.find('l');      // return 2
s.find('l', 3);   // return 3
s.find("ll");     // return 2
s.find("ll", 3);  // return s.npos
```



### ifind

```cpp
size_t ifind(char c, size_t pos=0) const;
size_t ifind(const char* s) const;
size_t ifind(const char* s, size_t pos) const
size_t ifind(const char* s, size_t pos, size_t n) const;
size_t ifind(const fastring& s, size_t pos=0) const;
size_t ifind(const std::string& s, size_t pos=0) const;
```

- v3.0.1 新增，与 [find](#find) 类似，但是忽略大小写。

- 示例

```cpp
fastring s("hello");
s.ifind('L');      // return 2
s.ifind('L', 3);   // return 3
s.ifind("Ll");     // return 2
```



### rfind

```cpp
1. size_t rfind(char c) const;
2. size_t rfind(char c, size_t pos) const;
3. size_t rfind(const char* s) const;
4. size_t rfind(const char* s, size_t pos) const;
5. size_t rfind(const char* s, size_t pos, size_t n) const;
6. size_t rfind(const fastring& s, size_t pos=npos) const;
7. size_t rfind(const std::string& s, size_t pos=npos) const;
```

- 与 [find](#find) 类似，只是从反向开始查找。

{{< hint info >}}
v3.0.1 新增上述 2, 4-7，并支持二进制字符串的反向查找。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
s.rfind('l');    // return 3
s.rfind('l', 2); // return 2
s.rfind("ll");   // return 2
s.rfind("le");   // return s.npos
```



### find_first_of

```cpp
1. size_t find_first_of(const char* s, size_t pos=0) const;
2. size_t find_first_of(const char* s, size_t pos, size_t n) const;
3. size_t find_first_of(const fastring& s, size_t pos=0) const;
4. size_t find_first_of(const std::string& s, size_t pos=0) const;
```

- 查找第一个出现的指定字符集 s 中的字符, n 是 s 的长度。
- 此方法从位置 pos(默认为0) 开始查找，遇到 s 中的任意字符时，即返回该字符的位置，否则返回 npos。

{{< hint info >}}
v3.0.1 新增 2-4，并支持二进制字符串的查找。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
s.find_first_of("def");    // return 1
s.find_first_of("ol", 3);  // return 3
```



### find_first_not_of

```cpp
1. size_t find_first_not_of(char c, size_t pos=0) const;
2. size_t find_first_not_of(const char* s, size_t pos=0) const;
3. size_t find_first_not_of(const char* s, size_t pos, size_t n) const;
4. size_t find_first_not_of(const fastring& s, size_t pos=0) const;
5. size_t find_first_not_of(const std::string& s, size_t pos=0) const;
```

- 查找第一个出现的非指定字符 c 或非指定字符集 s 中的字符，n 是 s 的长度。
- 此方法从位置 pos(默认为0) 开始查找，遇到不是 c 或非 s 中的任意字符时，即返回该字符的位置，否则返回 npos。

{{< hint info >}}
v3.0.1 中新增 3-5，并支持二进制字符串的查找。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
s.find_first_not_of("he");     // return 2
s.find_first_not_of("he", 3);  // return 3
s.find_first_not_of('x');      // return 0
```



### find_last_of

```cpp
1. size_t find_last_of(const char* s, size_t pos=npos) const;
2. size_t find_last_of(const char* s, size_t pos, size_t n) const;
3. size_t find_last_of(const fastring& s, size_t pos=npos) const;
4. size_t find_last_of(const std::string& s, size_t pos=npos) const;
```

- 查找最后一个出现的指定字符集 s 中的字符。
- 此方法从 pos(默认值为npos) 处开始反向查找，遇到 s 中的任意字符时，即返回该字符的位置，否则返回 `npos`。

{{< hint info >}}
v3.0.1 中新增 2-4。此方法支持二进制字符串的查找。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
s.find_last_of("le");     // return 3
s.find_last_of("le", 1);  // return 1
```



### find_last_not_of

```cpp
1. size_t find_last_not_of(char c, size_t pos=npos) const;
2. size_t find_last_not_of(const char* s, size_t pos=npos) const;
3. size_t find_last_not_of(const char* s, size_t pos, size_t n) const;
4. size_t find_last_not_of(const fastring& s, size_t pos=npos) const;
5. size_t find_last_not_of(const std::string& s, size_t pos=npos) const;
```

- 查找最后一个出现的非指定字符 c 或者非指定字符集 s 中的字符。
- 此方法从 pos(默认为npos) 处开始反向查找，遇到不等于 c 或者非 s 中的任意字符时，即返回该字符的位置，否则返回 `npos`。

{{< hint info >}}
v3.0.1 中新增 3-5，并且支持二进制字符串的查找。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
s.find_last_not_of("le");     // return 4
s.find_last_not_of("le", 3);  // return 0
s.find_last_not_of('o');      // return 3
```



### npos

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



### substr

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
### starts_with

```cpp
bool starts_with(char c) const;
bool starts_with(const char* s, size_t n) const;
bool starts_with(const char* s) const;
bool starts_with(const fastring& s) const;
bool starts_with(const std::string& s) const;
```

- 此方法判断 fastring 是否以字符 c 或字符串 s 开头，n 是 s 的长度。
- 当 s 为空字符串时，此方法始终返回 true。



### ends_with

```cpp
bool ends_with(char c) const;
bool ends_with(const char* s, size_t n) const;
bool ends_with(const char* s) const;
bool ends_with(const fastring& s) const;
bool ends_with(const std::string& s) const;
```

- 此方法判断 fastring 是否以字符 c 或字符串 s 结尾，n 是 s 的长度。
- 当 s 为空字符串时，此方法始终返回 true。



### match

```cpp
bool match(const char* pattern) const;
```

- 判断 fastring 是否匹配模式 pattern，`*` 匹配 0 个或多个字符，`?` 匹配单个字符。

- 示例

```cpp
fastring s("hello");
s.match("he??o");  // true
s.match("h*o");    // true
s.match("he?o");   // false
s.match("*o");     // true
s.match("h*l?");   // true
```



### remove_prefix

```cpp
fastring& remove_prefix(const char* s, size_t n);
fastring& remove_prefix(const char* s);
fastring& remove_prefix(const fastring& s);
fastring& remove_prefix(const std::string& s);
```

- 删除 fastring 中的前缀 s，n 是 s 的长度。
- v3.0.1 新增。

- 示例

```cpp
fastring s("12345678");
s.remove_prefix("123"); // s -> "45678"
s.remove_prefix("xxx"); // nothing will be done
```


### remove_suffix

```cpp
fastring& remove_suffix(const char* s, size_t n);
fastring& remove_suffix(const char* s);
fastring& remove_suffix(const fastring& s);
fastring& remove_suffix(const std::string& s);
```

- 删除 fastring 的后缀 s，n 是 s 的长度。

{{< hint warning >}}
v3.0.1 中将 **remove_tail** 更名为 **remove_suffix**。
{{< /hint >}}

- 示例

```cpp
fastring s("hello.log");
s.remove_suffix(".log"); // s -> "hello"
s.remove_suffix(".xxx"); // nothing will be done
```



### ———————————
### replace

```cpp
1. fastring& replace(const char* sub, const char* to, size_t maxreplace=0);
2. fastring& replace(const fastring& sub, const fastring& to, size_t maxreplace=0);
3. fastring& replace(const char* sub, size_t n, const char* to, size_t m, size_t maxreplace=0);
```

- 此方法将 fastring 中的子串 sub 替换成 to，参数 maxreplace 指定最大的替换次数，0 表示不限次数。n 和 m 分别是子串 sub、to 的长度。

{{< hint info >}}
v3.0.1 中新增 2-3，并支持二进制字符串的替换。
{{< /hint >}}

- 示例

```cpp
fastring s("hello");
s.replace("ll", "rr");                     // s -> "herro"
s.replace("err", "ok").replace("k", "o");  // s -> "hooo"
```



### strip

```cpp
template<typename ...X>
fastring& strip(X&& ...x);
```

- 修剪字符串，与 [trim](#trim) 等价。



### trim

```cpp
1. fastring& trim(char c, char d='b');
2. fastring& trim(unsigned char c, char d='b');
3. fastring& trim(signed char c, char d='b');
4. fastring& trim(const char* s=" \t\r\n", char d='b');
5. fastring& trim(size_t n, char d='b');
6. fastring& trim(int n, char d='b');
```

- 修剪字符串，去掉 fastring 左边、右边或两边的指定字符。
- 参数 d 表示方向，'l' 或 'L' 表示左边，'r' 或 'R' 表示右边，默认为 'b' 表示左右两边。
- 1-3, 去掉左、右或两边的单个字符 c。
- 4, 去掉左、右或两边的 s 中的字符。
- 5-6, 去掉左、右或两边的 n 个字符。

{{< hint info >}}
从 v3.0.1 开始，参数 `c` 可以是 `\0`。
{{< /hint >}}

- 示例

```cpp
fastring s = " sos\r\n";
s.trim();          // s -> "sos"
s.trim('s', 'l');  // s -> "os"
s.trim('s', 'r');  // s -> "o"

s = "[[xx]]";
s.trim(2);         // s -> "xx"
```



### tolower

```cpp
fastring& tolower();
```

- 此方法将 fastring 转换成小写，并返回 fastring 的引用。



### toupper

```cpp
fastring& toupper();
```

- 此方法将 fastring 转换成大写，并返回 fastring 的引用。



### lower

```cpp
fastring lower() const;
```

- 此方法返回 fastring 的小写形式。



### upper

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

