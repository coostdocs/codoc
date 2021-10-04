---
weight: 2
title: "fastring"
---

include: [co/fastring.h](https://github.com/idealvin/co/blob/master/include/co/fastring.h).


## fastring

`fastring` is the string type in co. It implements most of the methods in std::string and also provides some methods that std::string does not support.



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

- The first is the default constructor, which creates an empty fastring without allocating any memory.
- The second constructor also creates an empty fastring, but uses the parameter cap to specify the initial capacity.
- The third constructor creates a fastring from the given byte sequence, and the parameter n is the sequence length.
- The fourth constructor creates a fastring from a null-terminated string.
- The fifth constructor creates a fastring from std::string.
- The sixth and seventh constructors initialize fastring to a string of n characters c.
- The eighth is the copy constructor, which will copy the internal memory.
- The ninth one is the move constructor, which does not copy the memory.


- Example

```cpp
fastring s;               // empty string, no memory allocation
fastring s(32);           // empty string, pre-allocated memory (capacity is 32)
fastring s("hello");      // initialize s to "hello"
fastring s("hello", 3);   // initialize s to "hel"
fastring s(88,'x');       // initialize s to 88 'x'
fastring s('x', 88);      // initialize s to 88 'x'
fastring t(s);            // copy construction
fastring x(std::move(s)); // Move construction, s itself becomes an empty string
```



### fastring::operator=

```cpp
fastring& operator=(const char* s);
fastring& operator=(const std::string& s);
fastring& operator=(const fastring& s);
fastring& operator=(fastring&& s) noexcept;
```

- Assign value of a string to fastring.

- Example

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



### ———————————
### fastring::back

```cpp
char& back() const;
```

- This method returns a reference to the last character in fastring.
- If fastring is empty, calling this method will cause undefined behavior.

- Example

```cpp
fastring s("hello");
char c = s.back(); // c ='o'
s.back() ='x';     // s -> "hellx"
```



### fastring::front

```cpp
char& front() const;
```

- This method returns the reference of the first character in fastring.
- If fastring is empty, calling this method will cause undefined behavior.
- Example

```cpp
fastring s("hello");
char c = s.front(); // c ='h'
s.front() ='x';     // s -> "xello"
```



### fastring::operator[]

```cpp
char& operator[](size_t n) const;
```

- This method returns the reference of the nth character in fastring.
- If n is out of a reasonable range, calling this method will cause undefined behavior.

- Example

```cpp
fastring s("hello");
char c = s[1]; // c ='e'
s[1] ='x';     // s -> "hxllo"
```



### ———————————
### fastring::capacity

```cpp
size_t capacity() const;
```

- This method returns the capacity of fastring.



### fastring::c_str

```cpp
const char* c_str() const;
```

- This method gets the equivalent C string.
- This method adds a '\0' to the end of fastring, it will not change the size and content of fastring, but it may cause internal reallocation of memory.



### fastring::data

```cpp
const char* data() const;
```

- This method is similar to c_str(), but it does not guarantee that the string ends with '\0'.



### fastring::empty

```cpp
bool empty() const;
```

- This method determines whether fastring is empty.



### fastring::size

```cpp
size_t size() const;
```

- This method returns the length of fastring.



### fastring::substr

```cpp
fastring substr(size_t pos) const;
fastring substr(size_t pos, size_t len) const;
```

- The first version returns the substring starting at position pos.
- The second version returns a substring of length len starting at position pos.

- Example

```cpp
fastring s("hello");
s.substr(2);    // "llo"
s.substr(2, 2); // "ll"
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

- The first version appends a byte sequence, and n is the sequence length.
- The second version appends a string ending with '\0', and s can be part of fastring that performs the append operation.
- The third version appends a fastring, and s can be the fastring itself that performs the append operation.
- The fourth version appends a std::string.
- The 5th and 6th versions appends n characters c.
- The 7th to 9th version appends a single character c.
- This method returns a fastring reference, multiple append operations can be written to one line.


- Example

```cpp
fastring s;
s.append('c');               // s -> "c"
s.append(2,'c');             // s -> "ccc"
s.append('c', 2);            // s -> "ccccc"
s.clear();
s.append('c').append(2,'x'); // s -> "cxx"
s.append(s.c_str() + 1);     // s -> "cxxxx"
s.append(s.data(), 3);       // s -> "cxxxxcxx"
```



### fastring::operator<<

```cpp
fastring& operator<<(const char* s);
fastring& operator<<(const std::string& s);
fastring& operator<<(const fastring& s);
template<typename T> fastring& operator<<(T v);
```

- In the first version, s is a string ending in '\0', and s can be part of the fastring that performs the `operator<<` operation.
- In the third version, s can be the fastring itself that performs the `operator<<` operation.
- In the fourth version, T can be any built-in type, such as bool, char, int, double, void*, etc.

- Example

```cpp
fastring s;
s << false;         // s -> "false"
s.clear();
s << "hello "<< 23; // s -> "hello 23"
s << s.c_str() + 6; // s -> "hello 2323"
s << '';            // s -> "hello 2323 "
s << s;             // s -> "hello 2323 hello 2323 "
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

- This method is equivalent to the [append()](#fastringappend) method.

- Example

```cpp
fastring s;
s += 'c';  // s -> "c"
s += "xx"; // s -> "cxx"
```



### ———————————
### fastring::find

```cpp
size_t find(char c) const;
size_t find(char c, size_t pos) const;
size_t find(const char* s) const;
size_t find(const char* s, size_t pos) const;
```

- The first version searches for the character c starting from position 0.
- The second version looks for the character c starting at position pos.
- The third version starts from position 0 to find the substring s, which is implemented internally based on `strstr()`, and does not apply to fastring containing '\0'.
- The fourth version is like the third but searchs from position pos.
- This method returns the position of the character or substring when the search succeeds, otherwise it returns `fastring::npos`.

- Example

```cpp
fastring s("hello");
s.find('l');     // return 2
s.find('l', 3);  // return 3
s.find("ll");    // return 2
s.find("ll", 3); // return s.npos
```



### fastring::rfind

```cpp
size_t rfind(char c) const;
size_t rfind(const char* s) const;
```

- The first version searches for a single character in reverse. It is based on `strrchr()` and is not applicable to fastring containing '\0'.
- The second version searches for a substring in reverse.
- This method returns the position of the character or substring when the search succeeds, otherwise it returns fastring::npos.

- Example

```cpp
fastring s("hello");
s.rfind('l');  // return 3
s.rfind("ll"); // return 2
s.rfind("le"); // return s.npos
```



### fastring::find_first_of

```cpp
size_t find_first_of(const char* s) const;
size_t find_first_of(const char* s, size_t pos) const;
```

- Find the first occurrence of a character in the specified character set.
- The first version is searched from position 0.
- The second version is searched from position pos.
- This method is implemented based on `strcspn()` and is not suitable for fastring that contains '\0'.
- This method searches from the beginning, and when it encounters any character in s, it returns the position of the character, otherwise it returns `fastring::npos`.

- Example

```cpp
fastring s("hello");
s.find_first_of("def");   // return 1
s.find_first_of("ol", 3); // return 3
```



### fastring::find_first_not_of

```cpp
size_t find_first_not_of(const char* s) const;
size_t find_first_not_of(const char* s, size_t pos) const;
size_t find_first_not_of(char s, size_t pos=0);
```

- Find the first occurrence of a character not in the specified character set.
- The first version is searched from position 0.
- The second version is searched from position pos.
- The third version of the character set is a single character, and s cannot be '\0'.
- This method is implemented based on `strspn` and is not suitable for fastring containing '\0'.
- This method searches from the beginning. When it encounters any character not in s, it returns the position of the character, otherwise it returns `fastring::npos`.

- Example

```cpp
fastring s("hello");
s.find_first_not_of("he");    // return 2
s.find_first_not_of("he", 3); // return 3
s.find_first_not_of('x');     // return 0
```



### fastring::find_last_of

```cpp
size_t find_last_of(const char* s, size_t pos=npos) const;
```

- Find the last occurrence of a character in the specified character set.
- The parameter pos in this method defaults to npos, which means searching from the end of fastring.
- This method starts a reverse search from pos, and when it encounters any character in s, it returns the position of the character, otherwise it returns `fastring::npos`.

- Example

```cpp
fastring s("hello");
s.find_last_of("le");    // return 3
s.find_last_of("le", 1); // return 1
```



### fastring::find_last_not_of

```cpp
size_t find_last_not_of(const char* s, size_t pos=npos) const;
size_t find_last_not_of(char s, size_t pos=npos) const;
```

- Find the last occurrence of a character not in the specified character set.
- The parameter pos in this method defaults to npos, which means searching from the end of fastring.
- In the second version, s is a single character, and s cannot be '\0'.
- This method starts a reverse search from pos, and when it encounters any character not in s, it returns the position of the character, otherwise it returns `fastring::npos`.

- Example

```cpp
fastring s("hello");
s.find_last_not_of("le");    // return 4
s.find_last_not_of("le", 3); // return 0
s.find_last_not_of('o');     // return 3
```



### fastring::npos

```cpp
static const size_t npos = (size_t)-1;
```

- npos is the maximum value of the size_t type.
- When npos is used as the length, it means until the end of the string.
- When npos is used as the return value, it means not found.

- Example

```cpp
fastring s("hello");
size_t r = s.find('x');
r == s.npos; // true
```



### ———————————
### fastring::replace

```cpp
fastring& replace(const char* sub, const char* to, size_t maxreplace=0);
```

- This method replaces the substring sub in fastring with to. The parameter maxreplace specifies the maximum number of replacements, and 0 means unlimited.
- This method returns the fastring reference, and multiple replace operations can be written in one line.

- Example

```cpp
fastring s("hello");
s.replace("ll", "rr");                    // s -> "herro"
s.replace("err", "ok").replace("k", "o"); // s -> "hooo"
```



### fastring::strip

```cpp
fastring& strip(const char* s=" \t\r\n", char d='b');
fastring& strip(char s, char d='b');
```

- Trim a string, removes specified characters on the left, right or both sides of fastring.
- The parameter s is the characters to be trimmed, the parameter d represents the direction, 'l' or 'L' for left, 'r' or 'R' for right, and the default is 'b' for both sides.
- The first version removes blank characters on both sides of fastring by default.
- In the second version, s is a single character, and s cannot be '\0'.

- Example

```cpp
fastring s = "sos\r\n";
s.strip();        // s -> "sos"
s.strip('s','l'); // s -> "os"
s.strip('s','r'); // s -> "o"
```



### fastring::tolower

```cpp
fastring& tolower();
```

- This method converts fastring to lowercase and returns a reference to fastring.



### fastring::toupper

```cpp
fastring& toupper();
```

- This method converts fastring to uppercase and returns a reference to fastring.



### fastring::lower

```cpp
fastring lower() const;
```

- This method returns the lowercase form of fastring.



### fastring::upper

```cpp
fastring upper() const;
```

- This method returns the uppercase form of fastring.


- Example

```cpp
fastring s("Hello");
fastring x = s.lower(); // x = "hello", s remains unchanged
fastring y = s.upper(); // x = "HELLO", s remains unchanged
s.tolower();            // s -> "hello"
s.toupper();            // s -> "HELLO"
```



### ———————————
### fastring::clear

```cpp
void clear();
```

- This method only sets the size of fastring to 0, and the capacity remains unchanged.



### fastring::ensure

```cpp
void ensure(size_t n);
```

- This method ensures that the remaining memory of fastring can hold at least n characters.



### fastring::reserve

```cpp
void reserve(size_t n);
```

- This method ensures that the capacity of fastring is at least n.
- When n is less than the original capacity, the capacity remains unchanged.



### fastring::resize

```cpp
void resize(size_t n);
```

- This method sets the size of fastring to n.
- When n is greater than the original size, this operation will expand size to n, but will not fill the expanded part with '\0'.

- Example

```cpp
fastring s("hello");
s.resize(3);   // s -> "hel"
s.resize(6);
char c = s[5]; // c is an uncertain random value
```



### fastring::safe_clear

```cpp
void safe_clear();
```

- Like the clear(), but it will fill in the internal memory with zeros.



### fastring::swap

```cpp
void swap(fastring& s) noexcept;
void swap(fastring&& s) noexcept;
```

- This method exchanges the contents of two fastrings, only the data pointer, capacity, and size are exchanged internally.

- Example

```cpp
fastring s("hello");
fastring x("world");
s.swap(x); // s -> "world", x -> "hello"
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

- This method determines whether fastring starts with s, and s is a single character or a string.
- When s is an empty string, this method always returns true.



### fastring::ends_with

```cpp
bool ends_with(char s) const;
bool ends_with(const char* s, size_t n) const;
bool ends_with(const char* s) const;
bool ends_with(const fastring& s) const;
bool ends_with(const std::string& s) const;
```

- This method determines whether fastring ends with s, and s is a single character or a string.
- When s is an empty string, this method always returns true.



### fastring::match

```cpp
bool match(const char* pattern) const;
```

- Check whether fastring matches the pattern, `*` matches any string, and `?` matches a single character.

- Example

```cpp
fastring s("hello");
s.match("he??o"); // true
s.match("h*o");   // true
s.match("he?o");  // false
s.match("*o");    // true
s.match("h*l?");  // true
```



### fastring::lshift

```cpp
fastring& lshift(size_t n);
```

- This method shifts fastring to the left by n characters, which means removing the first n characters.

- Example

```cpp
fastring s("hello");
s.lshift(2); // s -> "llo"
s.lshift(8); // s -> ""
```



### fastring::remove_tail

```cpp
fastring& remove_tail(const char* s, size_t n);
fastring& remove_tail(const char* s);
fastring& remove_tail(const fastring& s);
fastring& remove_tail(const std::string& s);
```

- This method deletes the string s at the end of fastring if the fastring ends with s.

- Example

```cpp
fastring s("hello.log");
s.remove_tail(".log"); // s -> "hello"
```



### fastring::shrink

```cpp
void shrink();
```

- This method releases the extra memory in fastring. 

- Example

```cpp
fastring s("hello");
s.reserve(32); // capacity -> 32
s.shrink();    // capacity -> 6
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

- At least one parameter of this method is fastring.

- Example

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

- This method determines whether two strings are equal, at least one parameter is fastring.



### operator!=

```cpp
bool operator!=(const fastring& a, const fastring& b);
bool operator!=(const fastring& a, const char* b);
bool operator!=(const char* a, const fastring& b);
bool operator!=(const fastring& a, const std::string& b);
bool operator!=(const std::string& a, const fastring& b);
```

- This method determines whether two strings are not equal, at least one parameter is fastring.



### operator<

```cpp
bool operator<(const fastring& a, const fastring& b);
bool operator<(const fastring& a, const char* b);
bool operator<(const char* a, const fastring& b);
bool operator<(const fastring& a, const std::string& b);
bool operator<(const std::string& a, const fastring& b);
```

- This method determines whether the string a is less than b, and at least one parameter is fastring.



### operator>

```cpp
bool operator>(const fastring& a, const fastring& b);
bool operator>(const fastring& a, const char* b);
bool operator>(const char* a, const fastring& b);
bool operator>(const fastring& a, const std::string& b);
bool operator>(const std::string& a, const fastring& b);
```

- This method determines whether the string a is greater than b, and at least one parameter is fastring.



### operator<=

```cpp
bool operator<=(const fastring& a, const fastring& b);
bool operator<=(const fastring& a, const char* b);
bool operator<=(const char* a, const fastring& b);
bool operator<=(const fastring& a, const std::string& b);
bool operator<=(const std::string& a, const fastring& b);
```

- This method determines whether the string a is less than or equal to b. At least one parameter is fastring.



### operator>=

```cpp
bool operator>=(const fastring& a, const fastring& b);
bool operator>=(const fastring& a, const char* b);
bool operator>=(const char* a, const fastring& b);
bool operator>=(const fastring& a, const std::string& b);
bool operator>=(const std::string& a, const fastring& b);
```

- This method determines whether the string a is greater than or equal to b, and at least one parameter is fastring.


- Example

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

- Example

```cpp
fastring s("xx");
std::cout << s << std::endl;
```
