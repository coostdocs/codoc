---
weight: 21
title: "Path"
---

include: [co/path.h](https://github.com/idealvin/co/blob/master/include/co/path.h).


## path


`co/path` is ported from [golang](https://github.com/golang/go/blob/master/src/path/path.go), the path separator must be `'/'` . 




### path::clean()


```cpp
fastring clean(const fastring& s);
```


- Return the shortest equivalent form of the path, the consecutive separators in the path will be removed.



- Example



```cpp
path::clean("./x//y/");   // return "x/y"
path::clean("./x/..");    // return "."
path::clean("./x/../.."); // return ".."
```




### path::join()


```cpp
fastring join(const fastring& s, const fastring& t);
template <typename ...S> fastring join(const S&... s);
```


- The first version concatenates two fastrings into a complete path.
- The second version concatenates any number of fastrings into a complete path.
- The returned result is a cleaned path by path::clean().



- Example



```cpp
path::join("x", "y", "z"); // return "x/y/z"
path::join("/x/", "y");    // return "/x/y"
```




### path::split()


```cpp
std::pair<fastring, fastring> split(const fastring& s);
```


- Divide the path into two parts, dir and file. If the path does not contain a separator, the dir part will be empty.
- The returned result satisfies the property `path = dir + file`.



- Example



```cpp
path::split("/");    // -> {"/", ""}
path::split("/a");   // -> {"/", "a"}
path::split("/a/b"); // -> {"/a/", "b"}
```




### path::dir()


```cpp
fastring dir(const fastring& s);
```


- Return the directory part of the path, the return value is a cleaned path by path::clean().



- Example



```cpp
path::dir("a");  // return "."
path::dir("a/"); // return "a"
path::dir("/");  // return "/"
path::dir("/a"); // return "/";
```




### path::base()


```cpp
fastring base(const fastring& s);
```


- Return the last element of the path.



- Example



```cpp
path::base("");     // return "."
path::base("/");    // return "/"
path::base("/a/");  // return "a" ignores the delimiter at the end
path::base("/a");   // return "a"
path::base("/a/b"); // return "b"
```




### path::ext()


```cpp
fastring ext(const fastring& s);
```


- Return the file extension.



- Example



```cpp
path::ext("/a.cc");  // return ".cc"
path::ext("/a.cc/"); // return ""
```
