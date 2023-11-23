---
weight: 4
title: "fastream"
---

include: [co/fastream.h](https://github.com/idealvin/coost/blob/master/include/co/fastream.h).


## fastream

**fastream** 用于取代 C++ 标准库中的 `std::ostringstream`。std::ostringstream 性能较差，实测比 snprintf 慢好几倍，fastream 在不同平台测试比 snprintf 快 10~30 倍左右。



### constructor

```cpp
1. constexpr fastream() noexcept;
2. explicit fastream(size_t cap);
3. fastream(fastream&& s) noexcept;
```

- 1, 默认构造函数，创建一个空的 fastream 对象，内部不会分配任何内存。
- 2, 用参数 cap 指定 fastream 的初始容量，即预分配 cap 字节的内存。
- 3, 移动构造函数，不会进行内存拷贝。

- 示例

```cpp
fastream s;               // 空对象, 未分配内存
fastream s(1024);         // 预分配 1k 内存
fastream x(std::move(s)); // 移动构造, s 变成空对象
```



### operator=

```cpp
fastream& operator=(fastream&& s) noexcept;
```

- fastream 只支持 move 赋值操作，s 的内容被转移到 fastream 中，s 自身变成空对象。

- 示例

```cpp
fastream s(32);
fastream x;
x = std::move(s);  // x capacity -> 32,  s -> empty
```



### ———————————
### back

```cpp
char& back();
const char& back() const;
```

- 此方法返回 fastream 中最后一个字符的引用。

{{< hint warning >}}
若 fastream 为空，调用此方法会导致未定义的行为。
{{< /hint >}}

- 示例

```cpp
fastream s;
s.append("hello");
char c = s.back();   // c = 'o'
s.back() = 'x';      // s -> "hellx"
```



### front

```cpp
char& front();
const char& front() const;
```

- 此方法返回 fastream 中第一个字符的引用。

{{< hint warning >}}
若 fastream 为空，调用此方法会导致未定义的行为。
{{< /hint >}}

- 示例

```cpp
fastream s;
s.append("hello");
char c = s.front();  // c = 'h'
s.front() = 'x';     // s -> "xello"
```



### operator[]

```cpp
char& operator[](size_t n);
const char& operator[](size_t n) const;
```

- 此方法返回 fastream 中第 n 个字符的引用。

{{< hint warning >}}
若 n 超出合理的范围，调用此方法会导致未定义的行为。
{{< /hint >}}

- 示例

```cpp
fastream s;
s.append("hello");
char c = s[1];       // c = 'e'
s[1] = 'x';          // s -> "hxllo"
```



### ———————————
### capacity

```cpp
size_t capacity() const noexcept;
```

- 此方法返回 fastream 的容量。



### c_str

```cpp
const char* c_str() const;
```

- 此方法获取等效的 C 风格字符串 (`\0`结尾)。

{{< hint warning >}}
通过 `c_str()` 访问的字符数组是只读的，不可进行写操作。
{{< /hint >}}




### data

```cpp
char* data() noexcept;
const char* data() const noexcept;
```

- 此方法与 [c_str()](#c_str) 类似，但不保证字符串以 '\0' 结尾。



### empty

```cpp
bool empty() const noexcept;
```

- 此方法判断 fastream 是否为空。



### size

```cpp
size_t size() const noexcept;
```

- 此方法返回 fastream 内部数据的长度。



### str

```cpp
fastring str() const;
```

- 此方法以 fastring 形式返回 fastream 内部数据的一份拷贝。

- 示例

```cpp
fastream s;
s.append("hello");
fastring x = s.str();  // x = "hello"
```




### ———————————
### clear

```cpp
1. void clear();
2. void clear(char c);
```

- 此方法仅将 fastream 的 size 置为 0，capacity 保持不变。
- 2 与 1 类似，只是 size 置为 0 前会用字符 c 填充内部内存。

{{< hint info >}}
2 是 v3.0.1 新增，可用于清除内存中的敏感信息。
{{< /hint >}}



### ensure

```cpp
void ensure(size_t n);
```

- 此方法确保 fastream 剩余的内存能容纳至少 n 个字符。



### reserve

```cpp
void reserve(size_t n);
```

- 此方法调整 fastream 的容量，确保容量至少是 n。
- 当 n 小于原来的容量时，则保持容量不变。



### reset

```cpp
void reset();
```

- v2.0.3 新增。清空 fastream 并释放内存。



### resize

```cpp
1. void resize(size_t n);
2. void resize(size_t n, char c);
```

- 此方法将 fastream 的 size 设置为 n。
- 当 n 大于原来的 size 时，此操作将 size 扩大到 n。**1 中扩展的部分，内容是未定义的；2 中会用字符 `c` 填充扩展的部分。**

- 示例

```cpp
fastream s;
s.append("hello");
s.resize(3);    // s -> "hel"
s.resize(6);
char c = s[5];  // c 是不确定的随机值

s.resize(3);
s.resize(6, 0);
c = s[5];       // c 是 '\0'
```



### swap

```cpp
void swap(fastream& s) noexcept;
void swap(fastream&& s) noexcept;
```

- 交换两个 fastream，仅交换内部指针、容量、大小。

- 示例

```cpp
fastream s(32);
fastring x(64);
s.swap(x);  // s: cap -> 64,  x: cap -> 32
```



### ———————————
### append

```cpp
1.  fastream& append(const void* s, size_t n);
2.  fastream& append(const char* s);
3.  fastream& append(const fastring& s);
4.  fastream& append(const std::string& s);
5.  fastream& append(const fastream& s);

6.  fastream& append(size_t n, char c);
7.  fastream& append(char c);
8.  fastream& append(signed char v)
9. fastream& append(unsigned char c);

10. fastream& append(uint16 v);
11. fastream& append(uint32 v);
12. fastream& append(uint64 v);
```

- 1, 追加长度为 n 的字节序列。
- 2-4, 追加字符串 s。
- 5, 追加 fastream，s 可以是进行 append 操作的 fastream 对象本身。
- 6, 追加 n 个字符 c。
- 7-9, 追加单个字符 c。
- 10-12, 等价于 `append(&v, sizeof(v))`。

{{< hint warn >}}
从 v3.0.1 开始，1-2 参数 s 可以与 fastream 内部内存重叠。  
v3.0.2 移除了 `fastream& append(char c, size_t n);`。
{{< /hint >}}

- 示例

```cpp
fastream s;
int32 i = 7;
char buf[8];

s.append("xx");      // s -> "xx
s.append(s);         // 追加自身, s -> "xxxx"
s.append(buf, 8);    // 追加 8 字节
s.append('c');       // 追加单个字符
s.append(100, 'c');  // 追加 100 个 'c'

s.append(&i, 4);     // 追加 4 字节
s.append(i);         // 追加 4 字节, 与上同
s.append((int16)23); // 追加 2 字节

s.append(s.c_str() + 1); // v3.0.1 支持
```



### append_nomchk

```cpp
fastream& append_nomchk(const void* s, size_t n);
fastream& append_nomchk(const char* s)
```

- 与 [append()](#append) 类似，但不会检查 s 是否与内部内存重叠。

{{< hint warning >}}
若 s 与 fastream 内部内存可能重叠，则不能使用此方法。
{{< /hint >}}



### cat

```cpp
template<typename X, typename ...V>
fastream& cat(X&& x, V&& ... v);
```

- v2.0.3 新增。将任意数量的元素连接到 fastream 中。
- 此方法调用 `operator<<` 操作，将参数中的元素逐个追加到 fastream 中。

- 示例

```cpp
fastream s;
s << "hello";
s.cat(' ', 23, "xx", false); // s -> "hello 23xxfalse"
```



### operator<<

```cpp
fastream& operator<<(bool v);

fastream& operator<<(char v);
fastream& operator<<(signed char v);
fastream& operator<<(unsigned char v);

fastream& operator<<(short v);
fastream& operator<<(unsigned short v);
fastream& operator<<(int v);
fastream& operator<<(unsigned int v);
fastream& operator<<(long v);
fastream& operator<<(unsigned long v);
fastream& operator<<(long long v);
fastream& operator<<(unsigned long long v);

fastream& operator<<(double v);
fastream& operator<<(float v);
fastream& operator<<(const dp::_fpt& v);

fastream& operator<<(const void* v);
fastream& operator<<(std::nullptr_t);

fastream& operator<<(const char* s);
fastream& operator<<(const signed char* s);
fastream& operator<<(const unsigned char* s);
fastream& operator<<(const fastring& s);
fastream& operator<<(const std::string& s);
fastream& operator<<(const fastream& s);
```

- 将 bool、char、整数类型、浮点数类型、指针类型、字符串类型的值格式化后追加到 fastream 中。
- `operator<<(const dp::_fpt&)` 用于格式化输出浮点数，可指定有效小数位数。

- 示例

```cpp
fastream s;
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
fastream s;
s << dp::_2(3.1415);   // "3.14
s << dp::_3(3.1415);   // "3.141"
s << dp::_n(3.14, 1);  // "3.1", 与 dp::_1(3.14) 等价
```
{{< /hint >}}

{{< hint info >}}
[co.log](../../log/) 基于 fastream 实现，因此用 co.log 打印日志时，也可以用上述方法控制浮点数的有效小数位数。
```cpp
LOG << dp::_2(3.1415);
```
{{< /hint >}}
