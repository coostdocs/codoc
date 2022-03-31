---
weight: 3
title: "字符流(fastream)"
---

include: [co/fastream.h](https://github.com/idealvin/co/blob/master/include/co/fastream.h).


## fastream

`fastream` 用于取代 C++ 标准库中的 `std::ostringstream`。std::ostringstream 性能较差，实测比 snprintf 慢好几倍，fastream 在不同平台测试比 snprintf 快 10~30 倍左右。



### fastream::fastream

```cpp
fastream() noexcept;
explicit fastream(size_t cap);
fastream(fastream&& s) noexcept;
```

- 第 1 个是默认构造函数，创建一个空的 fastream 对象，内部不会分配任何内存。
- 第 2 个构造函数用参数 cap 指定 fastream 的初始容量，即预分配 cap 字节的内存。
- 第 3 个是 move 构造函数，不会进行内存拷贝。


- 示例

```cpp
fastream s;               // 状态为空的 fastream, 未分配内存
fastream s(1024);         // 预分配 1k 内存
fastream x(std::move(s));  // move 构造函数, s 变成空对象
 ```



### fastream::operator=

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
### fastream::back

```cpp
char& back() const;
```

- 此方法返回 fastream 中最后一个字符的引用。
- 若 fastream 为空，调用此方法会导致未定义的行为。

- 示例

```cpp
fastream s;
s.append("hello");
char c = s.back();   // c = 'o'
s.back() = 'x';      // s -> "hellx"
```



### fastream::front

```cpp
char& front() const;
```

- 此方法返回 fastream 中第一个字符的引用。
- 若 fastream 为空，调用此方法会导致未定义的行为。

- 示例

```cpp
fastream s;
s.append("hello");
char c = s.front();  // c = 'h'
s.front() = 'x';     // s -> "xello"
```



### fastream::operator[]

```cpp
char& operator[](size_t n) const;
```

- 此方法返回 fastream 中第 n 个字符的引用。
- 若 n 超出合理的范围，调用此方法会导致未定义的行为。

- 示例

```cpp
fastream s;
s.append("hello");
char c = s[1];       // c = 'e'
s[1] = 'x';          // s -> "hxllo"
```



### ———————————
### fastream::capacity

```cpp
size_t capacity() const;
```

- 此方法返回 fastream 的容量。



### fastream::c_str

```cpp
const char* c_str() const;
```

- 此方法获取等效的 C 字符串。
- 此方法在 fastream 末尾加上一个 '\0'，它不会改变 fastream 的 size 及内容，但有可能导致内部重新分配内存。



### fastream::data

```cpp
const char* data() const;
```

- 此方法与 c_str() 类似，但不保证字符串以 '\0' 结尾。



### fastream::empty

```cpp
bool empty() const;
```

- 此方法判断 fastream 是否为空。



### fastream::size
```cpp
size_t size() const;
```

- 此方法返回 fastream 内部数据的长度。



### fastream::str

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
### fastream::append

```cpp
1.  fastream& append(const void* s, size_t n);
2.  fastream& append(const char* s);
3.  fastream& append(const fastring& s);
4.  fastream& append(const std::string& s);
5.  fastream& append(const fastream& s);
6.  fastream& append(size_t n, char c);
7.  fastream& append(char c, size_t n);
8.  fastream& append(char c);
9.  fastream& append(signed char v)
10. fastream& append(unsigned char c);
11. fastream& append(short v);
12. fastream& append(unsigned short v);
13. fastream& append(int v);
14. fastream& append(unsigned int v);
15. fastream& append(long v);
16. fastream& append(unsigned long v);
17. fastream& append(long long v);
18. fastream& append(unsigned long long v);
```

- 第 1 个版本追加长度为 n 的字节序列。
- 第 2 个版本追加 C 风格字符串，与 fastring 不同，fastream 不检测内存是否重叠，s 不能是进行 append 操作的 fastream 的一部分。
- 第 3 个与第 4 个版本分别追加 fastring 与 std::string。
- 第 5 个版本追加 fastream，s 可以是进行 append 操作的 fastream 对象本身。
- 第 6 个与第 7 个版本追加 n 个字符 c。
- 第 8 到 10 个版本追加单个字符 c。
- 第 11 到 18 个版本以二进制形式追加 8 种内置整数类型，等价于 `append(&v, sizeof(v))`。


- 示例

```cpp
fastream s;
int32 i = 7;
char buf[8];

s.append("xx");        // 追加 C 字符串
s.append(s);           // 追加自身, s -> "xxxx"
s.append(buf, 8);      // 追加 8 字节
s.append('c');         // 追加单个字符
s.append(100, 'c');    // 追加 100 个 'c'
s.append('c', 100);    // 追加 100 个 'c'

s.append(&i, 4);       // 追加 4 字节
s.append(i);           // 追加 4 字节, 与上同
s.append((int16)23);   // 追加 2 字节

// 下面的用法是错误的, 不安全
s.append(s.c_str() + 1);
```



### fastream::cat

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



### fastream::operator<<

```cpp
fastream& operator<<(const signed char* s);
fastream& operator<<(const unsigned char* s);
template<typename T> fastream& operator<<(T&& t);
```

- 第 1, 2 个版本 v2.0.3 中新增，等价于 `fastream& operator<<(const char* s)`。
- 第 3 个版本中，T 可以是任意的基本类型(bool, char, int, double, void* 等)，以及字符串类型(const char*, fastring, std::string) 或者 fastream 类型。
- 与 fastring 不一样，fastream **不会进行内存安全检查**，像 `s << s.c_str() + 3;` 这样的操作是不安全的。

- 示例

```cpp
fastream s;
s << false;           // s -> "false"
s.clear();
s << "hello " << 23;  // s -> "hello 23"
s << ' ';             // s -> "hello 23 "
s << s;               // s -> "hello 23 hello 23 "
```



### ———————————
### fastream::clear

```cpp
void clear();
```

- 此方法仅将 fastream 的 size 置为 0，capacity 保持不变。



### fastream::ensure

```cpp
void ensure(size_t n);
```

- 此方法确保 fastream 剩余的内存容量能容纳至少 n 个字符。



### fastream::reserve

```cpp
void reserve(size_t n);
```

- 此方法调整 fastream 的容量，确保容量至少是 n。
- 当 n 小于原来的容量时，则保持容量不变。



### fastream::reset

```cpp
void reset();
```

- v2.0.3 新增。清空 fastream 并释放内存。



### fastream::resize

```cpp
void resize(size_t n);
```

- 此方法将 fastream 的 size 设置为 n。
- 当 n 大于原来的 size 时，此操作将 size 扩大到 n，但不会用 '\0' 填充扩展的部分。

- 示例

```cpp
fastream s;
s.append("hello");
s.resize(3);    // s -> "hel"
s.resize(6);
char c = s[5];  // c 是不确定的随机值
```



### fastream::safe_clear

```cpp
void safe_clear();
```

- 与 `clear()` 类似，但会将内部内存清零。



### fastream::swap

```cpp
void swap(fastream& s) noexcept;
void swap(fastream&& s) noexcept;
```

- 此方法交换两个 fastream，仅交换内部指针、容量、大小。

- 示例

```cpp
fastream s(32);
fastring x(64);
s.swap(x);  // s: cap -> 64,  x: cap -> 32
```



### ———————————
### 与 fastring 的互操作

`fastream` 与 `fastring` 都是继承自 `fast::stream`，它们的内存结构完全相同，因此可以方便的互相转换：

```cpp
fastream s;
s.append("Hello");
((fastring*)&s)->tolower();  // s -> "hello"

fastring x;
void f(fastream&);
f(*(fastream*)&x);
```

前面说到 fastream 的 append 操作不会检测内存重叠的情况，若有必要，可以转换成 fastring 再操作：

```cpp
fastream s;
s.append("hello");
((fastring*)&s)->append(s.c_str() + 1);
```

