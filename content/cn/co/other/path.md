---
weight: 11
title: "文件路径(path)"
---

include: [co/path.h](https://github.com/idealvin/coost/blob/master/include/co/path.h).


## path

此部分功能移植于 [golang](https://github.com/golang/go/blob/master/src/path/path.go)，路径分隔符必须为 `/`。



### path::clean

```cpp
fastring clean(const char* s, size_t n);
fastring clean(const char* s);
fastring clean(const fastring& s);
```

- 返回路径的最短等价形式，路径中连续的分隔符会被清除掉。

- 示例

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

- 将任意数量的字符串拼接成一个完整的路径，返回 [path::clean](#pathclean) 处理后的结果。
- 参数中的空字符串将会被忽略。

- 示例

```cpp
path::join("", "");         // ""
path::join("x", "y", "z");  // "x/y/z"
path::join("/x/", "y");     // "/x/y"
```



### path::split

```cpp
std::pair<fastring, fastring> split(const char* s, size_t n);
std::pair<fastring, fastring> split(const char* s);
std::pair<fastring, fastring> split(const fastring& s);
```

- 将路径切分为 dir, file 两部分，若路径中不含分隔符，则 dir 部分为空。
- 返回结果满足性质 `path = dir + file`。

- 示例

```cpp
path::split("/");     // -> { "/", "" }
path::split("/a");    // -> { "/", "a" }
path::split("/a/");   // -> { "/a/", "" }
path::split("/a/b");  // -> { "/a/", "b" }
```



### path::dir

```cpp
fastring dir(const char* s, size_t n);
fastring dir(const char* s);
fastring dir(const fastring& s);
```

- 返回路径的目录部分，返回值是 [path::clean](#pathclean) 处理后的结果。

- 示例

```cpp
path::dir("a");      // "."
path::dir("a/");     // "a"
path::dir("/");      // "/"
path::dir("/a");     // "/";
path::dir("/a/");    // "/a";
```



### path::base

```cpp
fastring base(const char* s, size_t n);
fastring base(const char* s);
fastring base(const fastring& s);
```

- 返回路径最后的一个元素。
- s 是空字符串时，返回 "."。
- s 中字符全是 `/` 时，返回 "/"。
- 其他情况，先将 s 末尾的 `/` 去掉。

- 示例

```cpp
path::base("");      // "."
path::base("/");     // "/"
path::base("/a/");   // "a"
path::base("/a");    // "a"
path::base("/a/b");  // "b"
```



### path::ext

```cpp
fastring ext(const char* s, size_t n);
fastring ext(const char* s);
fastring ext(const fastring& s);
```

- 返回路径中的文件扩展名。

- 示例

```cpp
path::ext("/a.c");   // ".c"
path::ext("a/b");    // ""
path::ext("/a.c/");  // ""
path::ext("a.");     // "."
```
