---
weight: 3
title: "fastream"
---

include: [co/fastream.h](https://github.com/idealvin/co/blob/master/include/co/fastream.h).


## fastream


`fastream` is used to replace `std::ostringstream` in the C++ standard library. The performance of std::ostringstream is poor which may be several times slower than snprintf, and fastream is about 10~30 times faster than snprintf on different platforms. 




### fastream::fastream


```cpp
fastream() noexcept;
explicit fastream(size_t cap);
fastream(fastream&& s) noexcept;
```


**Constructor**


- The first one is the default constructor, which creates an empty fastream object without any internal memory allocation.
- The second constructor uses the parameter cap to specify the initial capacity of fastream, that is, pre-allocated cap bytes of memory.
- The third is the move constructor, which does not copy memory.



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
fastream& append(const void* s, size_t n);
fastream& append(const char* s);
fastream& append(const fastring& s);
fastream& append(const std::string& s);
fastream& append(const fastream& s);
fastream& append(size_t n, char c);
fastream& append(char c, size_t n);
fastream& append(char c);
fastream& append(unsigned char c);
fastream& append(short v);
fastream& append(unsigned short v);
fastream& append(int v);
fastream& append(unsigned int v);
fastream& append(long v);
fastream& append(unsigned long v);
fastream& append(long long v);
fastream& append(unsigned long long v);
```


**Append operation**


- The first version appends a byte sequence of length n.
- The second version appends a C string. Unlike fastring, fastream does not check whether the memory overlaps, and s cannot be part of the fastream performing the append operation.
- The 3rd and 4th versions appends fastring and std::string respectively.
- The fifth version appends fastream, s can be the fastream object itself performing the append operation.
- The 6th and 7th versions appends n characters c.
- The 8th and 9th versions appends a single character c.
- The 10th to 17th versions appends 8 built-in integer types in binary form, which is equivalent to `append(&v, sizeof(v))`.



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




### fastream::operator<<


```cpp
fastream& operator<<(const char* s);
fastream& operator<<(const std::string& s);
fastream& operator<<(const fastring& s);
fastream& operator<<(const fastream& s)
template<typename T> fastream& operator<<(T v);
```


**Streaming output operation**


- The first 4 versions are equivalent to `append(s)`.
- In the fifth version, T can be any built-in data type, such as bool, char, int, double, void* and so on.
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
