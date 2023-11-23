---
weight: 4
title: "fastream"
---

include: [co/fastream.h](https://github.com/idealvin/coost/blob/master/include/co/fastream.h).


## fastream

**fastream** is used to replace `std::ostringstream` in the C++ standard library. The performance of std::ostringstream is poor which may be several times slower than `snprintf`, and fastream is about 10~30 times faster than snprintf on different platforms. 



### constructor

```cpp
1. constexpr fastream() noexcept;
2. explicit fastream(size_t cap);
3. fastream(fastream&& s) noexcept;
```

- 1, the default constructor, which creates an empty fastream object without any memory allocation.
- 2, uses the parameter `cap` to specify the initial capacity of fastream.
- 3, the move constructor.

- Example

```cpp
fastream s;              // empty fastream, no memory allocated
fastream s(1024);        // Pre-allocate 1k bytes
fastream x(std::move(s); // move construction, s becomes empty
```



### operator=

```cpp
fastream& operator=(fastream&& s) noexcept;
```

- Move assignment, the content of s is transferred to fastream, and s itself becomes empty.

- Example

```cpp
fastream s(32);
fastream x;
x = std::move(s); // x capacity -> 32, s -> empty
```



### ———————————
### back

```cpp
char& back();
const char& back() const;
```

- This method returns a reference to the last character in fastream.

{{< hint warning >}}
If fastream is empty, calling this method will cause undefined behavior.
{{< /hint >}}

- Example

```cpp
fastream s;
s.append("hello");
char c = s.back(); // c ='o'
s.back() ='x';     // s -> "hellx"
```



### front

```cpp
char& front();
const char& front() const;
```

- This method returns the reference of the first character in fastream.

{{< hint warning >}}
If fastream is empty, calling this method will cause undefined behavior.
{{< /hint >}}

- Example

```cpp
fastream s;
s.append("hello");
char c = s.front(); // c ='h'
s.front() ='x';     // s -> "xello"
```



### operator[]

```cpp
char& operator[](size_t n);
const char& operator[](size_t n) const;
```

- This method returns the reference of the nth character in fastream.

{{< hint warning >}}
If `n` is out of a reasonable range, calling this method will cause undefined behavior.
{{< /hint >}}

- Example

```cpp
fastream s;
s.append("hello");
char c = s[1]; // c ='e'
s[1] ='x';     // s -> "hxllo"
```



### ———————————
### capacity

```cpp
size_t capacity() const noexcept;
```

- This method returns the capacity of fastream.



### c_str

```cpp
const char* c_str() const;
```

- This method gets the equivalent C-style string (null terminated).

{{< hint warning >}}
Writing to the character array accessed through `c_str()` is undefined behavior.
{{< /hint >}}


### data

```cpp
char* data() noexcept;
const char* data() const noexcept;
```

- This method is similar to [c_str()](#c_str), but it does not guarantee that the string ends with '\0'.



### empty

```cpp
bool empty() const noexcept;
```

- This method determines whether fastream is empty.



### size

```cpp
size_t size() const noexcept;
```

- This method returns the length of data in fastream.



### str

```cpp
fastring str() const;
```

- This method returns a copy of fastream's internal data in the form of fastring.

- Example

```cpp
fastream s;
s.append("hello");
fastring x = s.str(); // x = "hello"
```



### ———————————
### clear

```cpp
1. void clear();
2. void clear(char c);
```

- This method sets the size of fastream to 0, and the capacity remains unchanged.
- The second is similar to the first, except that it will fill the memory with character `c` before setting its size to 0.

{{< hint info >}}
The second one is added in v3.0.1, which can be used to clear sensitive information in memory.
{{< /hint >}}



### ensure

```cpp
void ensure(size_t n);
```

- This method ensures that the remaining memory of fastream can hold at least `n` characters.



### reserve

```cpp
void reserve(size_t n);
```

- This method ensures that the capacity of fastream is at least `n`.
- When `n` is less than the original capacity, the capacity remains unchanged.



### reset

```cpp
void reset();
```

- Added in v2.0.3. Clear fastream and free the memory.



### resize

```cpp
1. void resize(size_t n);
2. void resize(size_t n, char c);
```

- This method sets the size of fastream to `n`.

- When `n` is greater than the original size, it will expand size to `n`. **In the 1st version, the content of the extended part is undefined. The 2nd version will fill the extended part with character `c`.**

- Example

```cpp
fastream s;
s.append("hello");
s.resize(3);    // s -> "hel"
s.resize(6);
char c = s[5];  // c is an uncertain random value

s.resize(3);
s.resize(6, 0);
c = s[5];       // c is '\0'
```



### swap

```cpp
void swap(fastream& s) noexcept;
void swap(fastream&& s) noexcept;
```


- Swap the contents of two fastreams, only the internal pointer, capacity, and size are exchanged.

- Example

```cpp
fastream s(32);
fastring x(64);
s.swap(x); // s: cap -> 64, x: cap -> 32
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

- 1, appends a byte sequence of length `n`.
- 2-4, append string `s`.
- 5, appends a fastream, `s` can be the fastream itself that performs the append operation.
- 6, append n characters `c`.
- 7-9, append a single character `c`.
- 10-12, equivalent to `append(&v, sizeof(v))`.

{{< hint warning >}}
Since v3.0.1, it is ok that the parameter `s` in 1 and 2 overlaps with the internal memory of `fastream`.  
v3.0.2 removed `fastream& append(char c, size_t n);`.
{{< /hint >}}

- Example

```cpp
fastream s;
int32 i = 7;
char buf[8];

s.append("xx");      // Append C string
s.append(s);         // append itself, s -> "xxxx"
s.append(buf, 8);    // Append 8 bytes
s.append('c');       // Append a single character
s.append(100,'c');   // Append 100'c'

s.append(&i, 4);     // Append 4 bytes
s.append(i);         // Append 4 bytes, same as above
s.append((int16)23); // Append 2 bytes

s.append(s.c_str() + 1); // ok since v3.0.1
```



### append_nomchk

```cpp
fastream& append_nomchk(const void* s, size_t n);
fastream& append_nomchk(const char* s)
```

- Similar to [append()](#append), but will not check if `s` overlaps with the internal memory.

{{< hint warning >}}
This method cannot be used if `s` may overlap with the internal memory of fastream.
{{< /hint >}}



### cat

```cpp
template<typename X, typename ...V>
fastream& cat(X&& x, V&& ... v);
```

- Added in v2.0.3. Concatenate any number of elements to fastream.
- This method appends elements in the parameters to fastream one by one through `operator<<`.

- Example

```cpp
fastream s;
s << "hello";
s.cat('', 23, "xx", false); // s -> "hello 23xxfalse"
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

- Format value of bool, char, integer type, floating point type, pointer type or string type, and append it to fastream.
- `operator<<(const dp::_fpt& v)` is used to output formatted floating-point number, which the effective decimal places can be specified.

- Example

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
**Specify the number of valid decimal places**  
coost provides `dp::_1, dp::_2, ..., dp::_16, dp::_n` to set the number of effective decimal places of floating-point numbers.
```cpp
fastream s;
s << dp::_2(3.1415);   // "3.14
s << dp::_3(3.1415);   // "3.141"
s << dp::_n(3.14, 1);  // "3.1", equivalent to dp::_1(3.14)
```
{{< /hint >}}

{{< hint info >}}
[co.log](../log/) is implemented based on fastream, so we can also use the above methods when printing logs.
```cpp
LOG << dp::_2(3.1415);
```
{{< /hint >}}
