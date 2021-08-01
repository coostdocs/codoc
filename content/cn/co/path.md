---
weight: 21
title: "文件路径(path)"
---


include: [co/path.h](https://github.com/idealvin/co/blob/master/include/co/path.h).


## path


`co/path` 移植于 [golang](https://github.com/golang/go/blob/master/src/path/path.go)，路径分隔符必须为 `'/'`。




### path::clean()
```cpp
fastring clean(const fastring& s);
```

- 返回路径的最短等价形式，路径中连续的分隔符会被清除掉。



- 示例
```cpp
path::clean("./x//y/");     // return "x/y"
path::clean("./x/..");      // return "."
path::clean("./x/../..");   // return ".."
```




### path::join()
```cpp
fastring join(const fastring& s, const fastring& t);
template <typename ...S> fastring join(const S&... s);
```

- 第 1 个版本将两个 fastring 字符串拼接成一个完整的路径。
- 第 2 个版本将任意数量的 fastring 字符串拼接成一个完整的路径。
- 返回的路径是 path::clean() 处理后的结果。



- 示例
```cpp
path::join("x", "y", "z");  // return "x/y/z"
path::join("/x/", "y");     // return "/x/y"
```




### path::split()
```cpp
std::pair<fastring, fastring> split(const fastring& s);
```

- 将路径切分为 dir, file 两部分，若路径中不含分隔符，则 dir 部分为空。
- 返回结果满足性质 `path = dir + file`。



- 示例
```cpp
path::split("/");     // -> { "/", "" }
path::split("/a");    // -> { "/", "a" }
path::split("/a/b");  // -> { "/a/", "b" }
```




### path::dir()
```cpp
fastring dir(const fastring& s);
```

- 返回路径的目录部分，返回值是 path::clean() 处理后的结果。



- 示例
```cpp
path::dir("a");      // return "."
path::dir("a/");     // return "a"
path::dir("/");      // return "/"
path::dir("/a");     // return "/";
```




### path::base()
```cpp
fastring base(const fastring& s);
```

- 返回路径最后的一个元素。



- 示例
```cpp
path::base("");      // return "."
path::base("/");     // return "/"
path::base("/a/");   // return "a"  忽略末尾的分隔符
path::base("/a");    // return "a"
path::base("/a/b");  // return "b"
```




### path::ext()
```cpp
fastring ext(const fastring& s);
```

- 返回文件扩展名。



- 示例
```cpp
path::ext("/a.cc");   // return ".cc"
path::ext("/a.cc/");  // return ""
```
