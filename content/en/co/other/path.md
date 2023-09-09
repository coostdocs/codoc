---
weight: 11
title: "Path"
---

include: [co/path.h](https://github.com/idealvin/coost/blob/master/include/co/path.h).


## path

This library is ported from golang's [path package](https://github.com/golang/go/blob/master/src/path/path.go), the path separator must be `/` . 



### path::clean

```cpp
fastring clean(const char* s, size_t n);
fastring clean(const char* s);
fastring clean(const fastring& s);
```

- Return the shortest equivalent form of the path, the consecutive separators in the path will be removed.

- Example

```cpp
path::clean("");            // "."
path::clean("./x//y/");     // "x/y"
path::clean("./x/..");      // "."
path::clean("./x/../..");   // ".."
```



### path::join

```cpp
template<typename ...S>
inline fastring join(S&&... s);
```

- Concatenate any number of strings into a complete path, the returned result is cleaned by [path::clean](#pathclean).
- Empty strings in the parameters will be ignored.

- Example

```cpp
path::join("", "");        // ""
path::join("x", "y", "z"); // "x/y/z"
path::join("/x/", "y");    // "/x/y"
```



### path::split

```cpp
std::pair<fastring, fastring> split(const char* s, size_t n);
std::pair<fastring, fastring> split(const char* s);
std::pair<fastring, fastring> split(const fastring& s);
```

- Divide the path into two parts, dir and file. If the path does not contain a separator, the dir part will be empty.
- The returned result satisfies the property `path = dir + file`.

- Example

```cpp
path::split("/");    // -> {"/", ""}
path::split("/a");   // -> {"/", "a"}
path::split("/a/");   // -> { "/a/", "" }
path::split("/a/b"); // -> {"/a/", "b"}
```



### path::dir

```cpp
fastring dir(const char* s, size_t n);
fastring dir(const char* s);
fastring dir(const fastring& s);
```

- Return the directory part of the path, the return value is a cleaned path by [path::clean](#pathclean).

- Example

```cpp
path::dir("a");    // "."
path::dir("a/");   // "a"
path::dir("/");    // "/"
path::dir("/a");   // "/";
path::dir("/a/");  // "/a";
```



### path::base

```cpp
fastring base(const char* s, size_t n);
fastring base(const char* s);
fastring base(const fastring& s);
```

- Return the last element of the path.
- If `s` is empty, return ".".
- If `s` consists entirely of slashes, return "/".
- In other cases, remove trailing slashes of `s` before extracting the last element.

- Example

```cpp
path::base("");     // "."
path::base("/");    // "/"
path::base("/a/");  // "a"
path::base("/a");   // "a"
path::base("/a/b"); // "b"
```



### path::ext

```cpp
fastring ext(const char* s, size_t n);
fastring ext(const char* s);
fastring ext(const fastring& s);
```

- Return the file name extension of the path.

- Example

```cpp
path::ext("/a.c");   // ".c"
path::ext("a/b");    // ""
path::ext("/a.c/");  // ""
path::ext("a.");     // "."
```
