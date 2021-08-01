---
weight: 4
title: "String utilities"
---

include: [co/str.h](https://github.com/idealvin/co/tree/master/include/co/str.h).


## split, strip, replace


### str::split


```cpp
std::vector<fastring> split(const char* s, char c, uint32 n=0);
std::vector<fastring> split(const fastring& s, char c, uint32 n=0);
std::vector<fastring> split(const char* s, const char* c, uint32 n=0);
std::vector<fastring> split(const fastring& s, const char* c, uint32 n=0);
```


- This function splits the string into several substrings, the original string remains unchanged, and returns the split result.
- Parameter s is a C string or fastring, parameter c is the separator, parameter n is the maximum number of splits, 0 or -1 means unlimited.
- In the fourth version, s cannot contain '\0', because the internal implementation uses `strstr()` to search for substrings.



- Example



```cpp
str::split("x y z", '');   // -> ["x", "y", "z"]
str::split("|x|y|",'|');   // -> ["", "x", "y"]
str::split("xooy", "oo");  // -> ["x", "y"]
str::split("xooy",'o');    // -> ["x", "", "y"]
str::split("xooy",'o', 1); // -> ["x", "oy"]
```




### str::strip


```cpp
fastring strip(const char* s, const char* c=" \t\r\n", char d='b');
fastring strip(const fastring& s, const char* c=" \t\r\n", char d='b');
fastring strip(const char* s, char c, char d ='b');
fastring strip(const fastring& s, char c, char d='b');
fastring strip(const fastring& s, const fastring& c, char d='b');
```


- This function removes the specified characters on the left or right sides of the string, the original string remains unchanged, and the result after strip is returned.
- The parameter s is a C string or fastring, the parameter c is the character set to be removed, the parameter d is the direction, 'l' or 'L' for left side, 'r' or 'R' for right side, and the default is 'b' means  both sides.
- The first and second versions remove the blank characters on both sides of the string by default.
- In the 3rd and 4th versions, c is a single character.



- Example



```cpp
str::strip(" xx\r\n");         // -> "xx"
str::strip("abxxa", "ab");     // -> "xx"
str::strip("abxxa", "ab",'l'); // -> "xxa"
str::strip("abxxa", "ab",'r'); // -> "abxx"
```




### str::replace


```cpp
fastring replace(const char* s, const char* sub, const char* to, uint32 n=0);
fastring replace(const fastring& s, const char* sub, const char* to, uint32 n=0);
```


- This function is used to replace substrings in the string, the original string remains unchanged, and the replaced result is returned.
- Parameter s is a C string or fastring, substring sub in s will be replaced with to. Parameter n is the maximum number of replacements, 0 or -1 means unlimited.
- In the second version, s cannot contain '\0', because the internal implementation uses `strstr()` to search for substrings.



- Example



```cpp
str::replace("xooxoox", "oo", "ee");    // -> "xeexeex"
str::replace("xooxoox", "oo", "ee", 1); // -> "xeexoox"
```




## Convert string to built-in type


### str::to_bool


```cpp
bool to_bool(const char* s);
bool to_bool(const fastring& s);
bool to_bool(const std::string& s);
```


- This function converts a string to bool type.
- When s is equal to "0" or "false", false is returned; when s is equal to "1" or "true", true is returned.
- **This functions returns false on error, and the errno will be EINVAL**.



- Example



```cpp
bool b = str::to_bool("true");  // x = true
bool x = str::to_bool("false"); // x = false
```




### str::to_double


```cpp
double to_double(const char* s);
double to_double(const fastring& s);
double to_double(const std::string& s);
```


- This function converts a string to double type.
- **This functions returns 0 on error, and the errno will be ERANGE or EINVAL**.



- Example



```cpp
double x = str::to_double("3.14"); // x = 3.14
```




### str::to_int


```cpp
int32 to_int32(const char* s);
int32 to_int32(const fastring& s);
int32 to_int32(const std::string& s);
int64 to_int64(const char* s);
int64 to_int64(const fastring& s);
int64 to_int64(const std::string& s);
uint32 to_uint32(const char* s);
uint32 to_uint32(const fastring& s);
uint32 to_uint32(const std::string& s);
uint64 to_uint64(const char* s);
uint64 to_uint64(const fastring& s);
uint64 to_uint64(const std::string& s);
```


- These functions convert a string to integer types.
- The parameter s can take an unit `k, m, g, t, p` at the end, which is not case sensitive.
- **These functions return 0 on error, and the errno will be ERANGE or EINVAL**.



- Example



```cpp
int32 i32;
int64 i64;
uint32 u32;
uint64 u64;

i32 = str::to_int32("-23");  // -23
u32 = str::to_uint32("4k");  // 4096
i64 = str::to_int32("8M");   // 8 << 20
i64 = str::to_int64("8T");   // 8ULL << 40
u64 = str::to_int64("1P");   // 1ULL << 50

i32 = str::to_int32("8g");
LOG << (i32 == 0);
LOG << (errno == ERANGE);

i32 = str::to_int32("abx");
LOG << (i32 == 0);
LOG << (errno == EINVAL);
```






## Convert built-in types to string


### str::from


```cpp
template<typename T>
inline fastring from(T t);
```


- This function converts built-in types to a string.
- T can be any built-in type, such as bool, int, double, void*, etc.



- Example



```cpp
fastring s;
s = str::from(true); // -> "true"
s = str::from(23);   // -> "23"
s = str::from(3.14); // -> "3.14"
```




## Convert STL container to debug string


### str::dbg


```cpp
template<typename T> fastring dbg(const std::vector<T>& v);
template<typename T> fastring dbg(const std::set<T>& v);
template<typename T> fastring dbg(const std::unordered_set<T>& v)
template<typename K, typename V> fastring dbg(const std::map<K, V>& v);
template<typename K, typename V> fastring dbg(const std::unordered_map<K, V>& v);
```


- This function converts a container to a debug string, which is generally for printing logs.



- Example



```cpp
std::vector<int> v {1, 2, 3 };
std::set<int> s {1, 2, 3 };
std::map<int, int> m {{1, 1}, {2, 2} };
str::dbg(v); // -> "[1,2,3]"
str::dbg(s); // -> "{1,2,3}"
str::dbg(m); // -> "{1:1,2:2}
```
