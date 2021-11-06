---
weight: 3
title: "fastream"
---

include: [co/fastream.h](https://github.com/idealvin/co/blob/master/include/co/fastream.h).


## fastream

`fastream` is used to replace `std::ostringstream` in the C++ standard library. The performance of std::ostringstream is poor which may be several times slower than `snprintf`, and fastream is about 10~30 times faster than snprintf on different platforms. 



### fastream::fastream

```cpp
fastream() noexcept;
explicit fastream(size_t cap);
fastream(fastream&& s) noexcept;
```

- The first one is the default constructor, which creates an empty fastream object without any memory allocation.
- The second constructor uses the parameter cap to specify the initial capacity of fastream.
- The third is the move constructor.

- Example

```cpp
fastream s;              // fastream with empty status, no memory allocated
fastream s(1024);        // Pre-allocate 1k memory
fastream x(std::move(s); // move constructor, s becomes an empty object
```



### fastream::operator=

```cpp
fastream& operator=(fastream&& s) noexcept;
```

- Move assignment, the content of s is transferred to fastream, and s itself becomes an empty object.

- Example

```cpp
fastream s(32);
fastream x;
x = std::move(s); // x capacity -> 32, s -> empty
```



### ———————————
### fastream::back

```cpp
char& back() const;
```

- This method returns a reference to the last character in fastream.
- If fastream is empty, calling this method will cause undefined behavior.

- Example

```cpp
fastream s;
s.append("hello");
char c = s.back(); // c ='o'
s.back() ='x';     // s -> "hellx"
```



### fastream::front

```cpp
char& front() const;
```

- This method returns a reference to the first character in fastream.
- If fastream is empty, calling this method will cause undefined behavior.

- Example

```cpp
fastream s;
s.append("hello");
char c = s.front(); // c ='h'
s.front() ='x';     // s -> "xello"
```



### fastream::operator[]

```cpp
char& operator[](size_t n) const;
```

- This method returns the reference of the nth character in fastream.
- If n is out of a reasonable range, calling this method will cause undefined behavior.

- Example

```cpp
fastream s;
s.append("hello");
char c = s[1]; // c ='e'
s[1] ='x';     // s -> "hxllo"
```



### ———————————
### fastream::capacity

```cpp
size_t capacity() const;
```

- This method returns the capacity of fastream.



### fastream::c_str


```cpp
const char* c_str() const;
```


- This method gets the equivalent C string.
- This method adds a '\0' to the end of fastream, it will not change the size and content of fastream, but may cause internal reallocation of memory.



### fastream::data

```cpp
const char* data() const;
```

- This method is similar to c_str(), but it does not guarantee that the string ends with '\0'.



### fastream::empty

```cpp
bool empty() const;
```

- This method determines whether fastream is empty.



### fastream::size

```cpp
size_t size() const;
```

- This method returns the length of data in fastream.



### fastream::str

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

- The first version appends a byte sequence of length n.
- The second version appends a C string. Unlike fastring, fastream does not check whether the memory overlaps, and s cannot be part of the fastream performing the append operation.
- The 3rd and 4th versions appends fastring and std::string respectively.
- The 5th version appends fastream, s can be the fastream performing the append operation.
- The 6th and 7th versions appends n characters c.
- The 8th to 10th versions appends a single character c.
- The 11th to 18th versions appends 8 built-in integer types in binary form, which is equivalent to `append(&v, sizeof(v))`.


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
s.append('c', 100);  // append 100'c'

s.append(&i, 4);     // Append 4 bytes
s.append(i);         // Append 4 bytes, same as above
s.append((int16)23); // Append 2 bytes

// The following usage is wrong and unsafe
s.append(s.c_str() + 1);
```



### fastream::cat

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



### fastream::operator<<

```cpp
fastream& operator<<(const signed char* s);
fastream& operator<<(const unsigned char* s);
template<typename T> fastream& operator<<(T&& t);
```

- The first two versions are added in v2.0.3, which are equivalent to `fastream& operator<<(const char* s)`.
- In the third version, T can be any basic type (bool, char, int, double, void*, etc.), string type (const char*, fastring, std::string) or fastream.
- Unlike fastring, fastream **does not perform memory security checks**, operations like `s << s.c_str() + 3;` are not safe.

- Example

```cpp
fastream s;
s << false;         // s -> "false"
s.clear();
s << "hello "<< 23; // s -> "hello 23"
s << '';            // s -> "hello 23 "
s << s;             // s -> "hello 23 hello 23 "
```



### ———————————
### fastream::clear

```cpp
void clear();
```

- This method only sets the size of fastream to 0, and the capacity remains unchanged.



### fastream::ensure

```cpp
void ensure(size_t n);
```

- This method ensures that the remaining memory of fastream can hold at least n characters.



### fastream::reset

```cpp
void reset();
```

- Added in v2.0.3. Clear fastream and free the memory.



### fastream::reserve

```cpp
void reserve(size_t n);
```

- This method adjusts the capacity of fastream to ensure that the capacity is at least n.



### fastream::resize

```cpp
void resize(size_t n);
```

- This method sets the size of fastream to n.
- When n is greater than the original size, this operation will expand size to n, but will not fill the expanded part with '\0'.

- Example

```cpp
fastream s;
s.append("hello");
s.resize(3);   // s -> "hel"
s.resize(6);
char c = s[5]; // c is an uncertain random value
```



### fastream::safe_clear

```cpp
void safe_clear();
```

- Like the clear(), but will fill the internal memory with zeros.



### fastream::swap

```cpp
void swap(fastream& s) noexcept;
void swap(fastream&& s) noexcept;
```


- This method exchanges the contents of two fastreams, only the internal pointer, capacity, and size are exchanged.

- Example

```cpp
fastream s(32);
fastring x(64);
s.swap(x); // s: cap -> 64, x: cap -> 32
```



### ———————————
### Interoperability with fastring

`fastream` and `fastring` are both inherited from `fast::stream`, the memory structure of them is exactly the same, so they can be easily converted to each other:

```cpp
fastream s;
s.append("Hello");
((fastring*)&s)->tolower(); // s -> "hello"

fastring x;
void f(fastream&);
f(*(fastream*)&x);
```

As mentioned earlier, the append operation of fastream will not check memory overlap. If necessary, it can be converted to fastring to perform the operation safely:

```cpp
fastream s;
s.append("hello");
((fastring*)&s)->append(s.c_str() + 1);
```

