---
weight: 3
title: "fastring"
---

include: [co/fastring.h](https://github.com/idealvin/coost/blob/master/include/co/fastring.h).


## fastring

**fastring** is the string type in coost. It implements most of the methods in `std::string` and also provides some methods that `std::string` does not support.


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

- 1, the default constructor, which creates an empty fastring without allocating any memory.
- 2, creates an empty fastring, but uses the parameter cap to specify the initial capacity.
- 3, creates a fastring from the given byte sequence, and the parameter `n` is the sequence length.
- 4, creates a fastring from a null-terminated string.
- 5, creates a fastring from `std::string`.
- 6-7, initialize fastring to a string of `n` characters `c`.
- 8, the copy constructor, which will copy the internal memory.
- 9, the move constructor, which does not copy the memory.


- Example

```cpp
fastring s;               // empty string, no memory allocation
fastring s(32);           // empty string, pre-allocated memory (capacity is 32)
fastring s("hello");      // initialize s to "hello"
fastring s("hello", 3);   // initialize s to "hel"
fastring s(88,'x');       // initialize s to 88 'x'
fastring s('x', 88);      // initialize s to 88 'x'
fastring t(s);            // copy construction
fastring x(std::move(s)); // Move construction, s itself becomes empty
```



### operator=

```cpp
1. fastring& operator=(const char* s);
2. fastring& operator=(const std::string& s);
3. fastring& operator=(const fastring& s);
4. fastring& operator=(fastring&& s) noexcept;
```

- 1, assigns a null-terminated string to fastring, `s` can be part of the fastring for assignment.
- 2, assigns the value of `std::string` to fastring.
- 3, the copy assignment operation.
- 4, the move assignment operation, `s` itself will become empty.

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



### assign

```cpp
1. fastring& assign(const void* s, size_t n);
2. template<typename S> fastring& assign(S&& s);
```

- Assign value to fastring, added in v3.0.1.
- In the first one, `n` is the length of `s`. The second is equivalent to [operator=](#operator).



### ———————————
### back

```cpp
char& back();
const char& back() const;
```

- This method returns a reference to the last character in fastring.

{{< hint warning >}}
If fastring is empty, calling this method will cause undefined behavior.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
char c = s.back(); // c ='o'
s.back() ='x';     // s -> "hellx"
```



### front

```cpp
char& front();
const char& front() const;
```

- This method returns the reference of the first character in fastring.

{{< hint warning >}}
If fastring is empty, calling this method will cause undefined behavior.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
char c = s.front(); // c ='h'
s.front() ='x';     // s -> "xello"
```



### operator[]

```cpp
char& operator[](size_t n);
const char& operator[](size_t n) const;
```

- This method returns the reference of the nth character in fastring.

{{< hint warning >}}
If `n`` is out of a reasonable range, calling this method will cause undefined behavior.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
char c = s[1]; // c ='e'
s[1] ='x';     // s -> "hxllo"
```



### ———————————
### capacity

```cpp
size_t capacity() const noexcept;
```

- This method returns the capacity of fastring.



### c_str

```cpp
const char* c_str() const;
```

- This method gets the equivalent C string.
- This method adds a '\0' to the end of fastring, it will not change the size and content of fastring, but it may cause internal reallocation of memory.



### data

```cpp
const char* data() const noexcept;
```

- This method is similar to [c_str()](#c_str), but it does not guarantee that the string ends with '\0'.



### empty

```cpp
bool empty() const noexcept;
```

- This method determines whether fastring is empty.



### size

```cpp
size_t size() const noexcept;
```

- This method returns the length of fastring.



### ———————————
### clear

```cpp
1. void clear();
2. void clear(char c);
```

- Set the size of fastring to 0, the capacity remains unchanged.
- The second is similar to the first, except that it will fill the memory with character `c` before setting its size to 0.

{{< hint info >}}
The second one is added in v3.0.1, which can be used to clear sensitive information in memory.
{{< /hint >}}



### ensure

```cpp
void ensure(size_t n);
```

- This method ensures that the remaining memory of fastring can hold at least `n` characters.



### reserve

```cpp
void reserve(size_t n);
```

- This method ensures that the capacity of fastring is at least `n`.
- When `n` is less than the original capacity, the capacity remains unchanged.



### reset

```cpp
void reset();
```

- Added in v2.0.3. Clear fastring and free the memory.



### resize

```cpp
void resize(size_t n);
```

- This method sets the size of fastring to `n`.

{{< hint warning >}}
When `n` is greater than the original size, it will expand size to `n`, but **will not fill the expanded part with zeroes**.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
s.resize(3);   // s -> "hel"
s.resize(6);
char c = s[5]; // c is an uncertain random value
```



### shrink

```cpp
void shrink();
```

- Release the extra memory in fastring. 

- Example

```cpp
fastring s("hello");
s.reserve(32); // capacity -> 32
s.shrink();    // capacity -> 6
```



### swap

```cpp
void swap(fastring& s) noexcept;
void swap(fastring&& s) noexcept;
```

- Swap the contents of two fastrings, only the data pointer, capacity, and size are exchanged internally.

- Example

```cpp
fastring s("hello");
fastring x("world");
s.swap(x); // s -> "world", x -> "hello"
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

- 1, appends a byte sequence of length `n`.
- 2, appends a null-terminated string, and `s` can be part of fastring that performs the append operation.
- 3, appends a fastring, and s can be the fastring itself that performs the append operation.
- 4, appends `std::string`.
- 5-6, appends `n` characters `c`.
- 7, appends a single character c.


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



### append_nomchk

```cpp
fastring& append_nomchk(const void* s, size_t n);
fastring& append_nomchk(const char* s)
```

- Similar to [append()](#append), but will not check if `s` overlaps with the internal memory.

{{< hint warning >}}
This method cannot be used if `s` may overlap with the internal memory of fastring.
{{< /hint >}}



### cat

```cpp
template<typename X, typename ...V>
fastring& cat(X&& x, V&& ... v);
```

- Added in v2.0.3. Concatenate any number of elements to fastring.
- This method appends elements in the parameters to fastring one by one through `operator<<`.

- Example

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

- Format value of bool, char, integer type, floating point type, pointer type or string type, and append it to fastring.
- `operator<<(const dp::_fpt& v)` is used to output formatted floating-point number, which the effective decimal places can be specified.

- Example

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
**Specify the number of valid decimal places**  
coost provides `dp::_1, dp::_2, ..., dp::_16, dp::_n` to set the number of effective decimal places of floating-point numbers.
```cpp
fastring s;
s << dp::_2(3.1415);   // "3.14
s << dp::_3(3.1415);   // "3.141"
s << dp::_n(3.14, 1);  // "3.1", equivalent to dp::_1(3.14)
```
{{< /hint >}}


### operator+=

```cpp
fastring& operator+=(const char* s);
fastring& operator+=(const fastring& s);
fastring& operator+=(const std::string& s);
fastring& operator+=(char c);
```

- This method is equivalent to the [append()](#append) method.

- Example

```cpp
fastring s;
s += 'c';  // s -> "c"
s += "xx"; // s -> "cxx"
```



### push_back

```cpp
fastring& push_back(char c);
```

- Append single character to fastring, equivalent to [append(c)](#append).



### pop_back

```cpp
char pop_back();
```

- Pop and return the last character of fastring.

{{< hint warning >}}
If fastring is empty, calling this method will cause undefined behavior.
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

- Compare fastring with the specified string.
- 1-4, compare fastring with `s`, `n` is the length of `s`.
- 5-8, Compare the `[pos, pos+len)` part of fastring with `s`, `n` is the length of `s`.
- 9-10, Compare the `[pos, pos+len)` part of fastring with the `[spos, spos+n)` part of `s`.



### contains

```cpp
bool contains(char c) const;
bool contains(const char* s) const;
bool contains(const fastring& s) const;
bool contains(const std::string& s) const;
```

- Check if the fastring contains the given character or substring.



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

- 1, search for character c from position 0.
- 2, search for character c from position `pos`.
- 3, added in v3.0, search for character c in range `[pos, pos + len)`.
- 4, search substring s from position 0.
- 5, search substring s from position `pos`.
- 6, search substring s from position `pos`, `n` is the length of s.
- 7-8, search substring s from position `pos`.
- This method returns the position of the character or substring when the search succeeds, otherwise it returns `npos`.

{{< hint info >}}
Coost v3.0.1 adds the above 6-8, and supports the search of binary strings.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
s.find('l');     // return 2
s.find('l', 3);  // return 3
s.find("ll");    // return 2
s.find("ll", 3); // return s.npos
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

- Added in v3.0.1, similar to [find](#find), but will ignore the case.

- Example

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

- Similar to [find](#find), but search in reverse order.

{{< hint info >}}
Coost v3.0.1 adds the above 2, 4-7, and supports reverse search of binary strings.
{{< /hint >}}

- Example

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

- Find the first occurrence of a character in the specified character set `s`, `n` is the length of `s`.
- This method searches from position `pos`(0 by default), and when it encounters any character in `s`, it returns the position of the character, otherwise it returns `npos`.

{{< hint info >}}
Coost v3.0.1 adds 2-4, and supports search of binary strings.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
s.find_first_of("def");   // return 1
s.find_first_of("ol", 3); // return 3
```



### find_first_not_of

```cpp
1. size_t find_first_not_of(char c, size_t pos=0) const;
2. size_t find_first_not_of(const char* s, size_t pos=0) const;
3. size_t find_first_not_of(const char* s, size_t pos, size_t n) const;
4. size_t find_first_not_of(const fastring& s, size_t pos=0) const;
5. size_t find_first_not_of(const std::string& s, size_t pos=0) const;
```

- Find the first occurrence of a character not equal to `c` or not in the specified character set `s`, from position `pos`, `n` is the length of `s`.
- This method searches from postion `pos`(0 by default), when it encounters any character not equal to `c` or not in `s`, it returns the position of the character, otherwise it returns `npos`.

{{< hint info >}}
Coost v3.0.1 adds 3-5, and supports search of binary strings.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
s.find_first_not_of("he");    // return 2
s.find_first_not_of("he", 3); // return 3
s.find_first_not_of('x');     // return 0
```



### find_last_of

```cpp
1. size_t find_last_of(const char* s, size_t pos=npos) const;
2. size_t find_last_of(const char* s, size_t pos, size_t n) const;
3. size_t find_last_of(const fastring& s, size_t pos=npos) const;
4. size_t find_last_of(const std::string& s, size_t pos=npos) const;
```

- Find the last occurrence of a character in the specified character set `s`.
- This method starts a reverse search from `pos`(`npos` by default), and when it encounters any character in `s`, it returns the position of the character, otherwise it returns `npos`.

{{< hint info >}}
Coost v3.0.1 adds 2-4. This method supports search of binary strings.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
s.find_last_of("le");    // return 3
s.find_last_of("le", 1); // return 1
```



### find_last_not_of

```cpp
1. size_t find_last_not_of(char c, size_t pos=npos) const;
2. size_t find_last_not_of(const char* s, size_t pos=npos) const;
3. size_t find_last_not_of(const char* s, size_t pos, size_t n) const;
4. size_t find_last_not_of(const fastring& s, size_t pos=npos) const;
5. size_t find_last_not_of(const std::string& s, size_t pos=npos) const;
```

- Find the last occurrence of a character not equal to `c` or not in the specified character set `s`.
- This method starts a reverse search from `pos`(`npos` by default), and when it encounters any character not equal to `c` or not in `s`, it returns the position of the character, otherwise it returns `npos`.

{{< hint info >}}
Coost v3.0.1 adds 3-5, and supports search of binary strings.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
s.find_last_not_of("le");    // return 4
s.find_last_not_of("le", 3); // return 0
s.find_last_not_of('o');     // return 3
```



### npos

```cpp
static const size_t npos = (size_t)-1;
```

- It is the maximum value of the size_t type.
- When it is used as the length, it means until the end of the string.
- When it is used as the return value, it means not found.

- Example

```cpp
fastring s("hello");
size_t r = s.find('x');
r == s.npos; // true
```



### substr

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
### starts_with

```cpp
bool starts_with(char c) const;
bool starts_with(const char* s, size_t n) const;
bool starts_with(const char* s) const;
bool starts_with(const fastring& s) const;
bool starts_with(const std::string& s) const;
```

- This method determines whether fastring starts with character `c` or string `s`, `n` is the length of `s`.
- When `s` is empty, this method always returns true.



### ends_with

```cpp
bool ends_with(char s) const;
bool ends_with(const char* s, size_t n) const;
bool ends_with(const char* s) const;
bool ends_with(const fastring& s) const;
bool ends_with(const std::string& s) const;
```

- This method determines whether fastring ends with character `c` or string `s`, `n` is the length of `s`.
- When `s` is empty, this method always returns true.



### match

```cpp
bool match(const char* pattern) const;
```

- Check whether fastring matches the pattern, `*` matches 0 or more characters, and `?` matches exactly one character.

- Example

```cpp
fastring s("hello");
s.match("he??o"); // true
s.match("h*o");   // true
s.match("he?o");  // false
s.match("*o");    // true
s.match("h*l?");  // true
```



### remove_prefix

```cpp
fastring& remove_prefix(const char* s, size_t n);
fastring& remove_prefix(const char* s);
fastring& remove_prefix(const fastring& s);
fastring& remove_prefix(const std::string& s);
```

- Remove the prefix `s` of fastring, `n` is the length of `s`.
- Added in v3.0.1.

- Example

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

- Remove the suffix `s` from fastring, `n` is the length of `s`.

{{< hint warning >}}
Coost v3.0.1 renames **remove_tail** to **remove_suffix**.
{{< /hint >}}

- Example

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

- This method replaces the substring `sub` in fastring with `to`. The parameter `maxreplace` specifies the maximum number of replacements, and 0 means unlimited. `n` and `m` are the lengths of the substrings `sub` and `to` respectively.

{{< hint info >}}
Coost v3.0.1 adds 2-3, and supports replacement of binary strings.
{{< /hint >}}

- Example

```cpp
fastring s("hello");
s.replace("ll", "rr");                    // s -> "herro"
s.replace("err", "ok").replace("k", "o"); // s -> "hooo"
```



### strip

```cpp
fastring& strip(const char* s=" \t\r\n", char d='b');
fastring& strip(char s, char d='b');
```

- Trim the string, the same as [trim](#trim).



### trim

```cpp
1. fastring& trim(char c, char d='b');
2. fastring& trim(unsigned char c, char d='b');
3. fastring& trim(signed char c, char d='b');
4. fastring& trim(const char* s=" \t\r\n", char d='b');
5. fastring& trim(size_t n, char d='b');
6. fastring& trim(int n, char d='b');
```

- Trim the string, remove the specified characters on the left, right, or both sides of fastring.
- The parameter `d` is the direction, 'l' or 'L' means left, 'r' or 'R' means right, and the default is 'b' for both sides.
- 1-3, remove single character `c` from left, right or both sides.
- 4, remove characters in `s` on the left, right or both sides.
- 5-6, remove `n` characters from the left, right or both sides.

{{< hint info >}}
Since v3.0.1, the parameter `c` can be `\0`.
{{< /hint >}}

- Example

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

- This method converts fastring to lowercase and returns a reference to fastring.



### toupper

```cpp
fastring& toupper();
```

- This method converts fastring to uppercase and returns a reference to fastring.



### lower

```cpp
fastring lower() const;
```

- This method returns the lowercase form of fastring.



### upper

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
