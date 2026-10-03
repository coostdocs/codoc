---
weight: 10
title: "文件路径"
---


## 头文件

```cpp
#include "co/path.h"
```

API 在 `path` 命名空间，不是 `co`。

移植自 Go 的 `path` 包，路径分隔符固定为 `/`。


## 概述

- `clean`：返回最短等价路径。
- `join`：连接多个路径元素。
- `split`：按最后一个 `/` 拆成 dir 和 file。
- `dir`：返回目录部分。
- `base`：返回最后一段。
- `ext`：返回扩展名。

所有返回 `co::string` 的函数都有三个重载：`(const char*, size_t)`、`(const char*)`、`(const co::string&)`。


## clean

```cpp
co::string path::clean(const char* s, size_t n);
```

返回最短等价路径。

```cpp
path::clean("")           // -> "."
path::clean(".//x/")      // -> "x"
path::clean("./x/../..")  // -> ".."
path::clean("/x/../..")   // -> "/"
path::clean("x//y//z")    // -> "x/y/z"
```


## join

```cpp
template<typename ...X>
co::string path::join(X&&... x);
```

连接任意数量的路径元素，结果会被 `clean`，空元素忽略。

```cpp
path::join("", "")       // -> ""
path::join("/x", "y")    // -> "/x/y"
path::join("/x/", "y")   // -> "/x/y"
```


## split

```cpp
std::pair<co::string, co::string> path::split(const char* s, size_t n);
```

按最后一个 `/` 拆成 dir 和 file，满足 `path = dir + file`。

```cpp
path::split("/a/")   // -> <"/a/", "">
path::split("/a/b")  // -> <"/a/", "b">
```


## dir

```cpp
co::string path::dir(const char* s, size_t n);
```

返回目录部分，结果会被 `clean`。

```cpp
path::dir("")     // -> "."
path::dir("a")    // -> "."
path::dir("/a")   // -> "/"
path::dir("/a/")  // -> "/a"
```


## base

```cpp
co::string path::base(const char* s, size_t n);
```

返回最后一段，先去尾部斜杠。

```cpp
path::base("")       // -> "."
path::base("/a/b")   // -> "b"
path::base("/a/b/")  // -> "b"
```

路径全为斜杠时返回 `"/"`。


## ext

```cpp
co::string path::ext(const char* s, size_t n);
```

返回文件扩展名（含 `.`）。

```cpp
path::ext("x/x.c")  // -> ".c"
path::ext("a/b")    // -> ""
path::ext("/b.c/")  // -> ""
path::ext("a.")     // -> "."
```


## 示例

```cpp
#include "co/path.h"
#include "co/print.h"

int main() {
    co::println(path::clean("./x/../.."));       // ..
    co::println(path::join("/x/", "y", "z"));    // /x/y/z

    auto p = path::split("/a/b");
    co::println(p.first, " | ", p.second);       // /a/ | b

    co::println(path::dir("/a/b"));              // /a
    co::println(path::base("/a/b/"));            // b
    co::println(path::ext("x/x.c"));             // .c
    return 0;
}
```

## 注意事项

- 路径分隔符固定为 `/`，不区分平台。
- `clean` 是其它函数的基础，结果都经过清理。
- `join` 忽略空元素。
- `split` 结果满足 `path = dir + file`。
- `base("")` 和 `dir("")` 返回 `"."`。
- 路径全为斜杠时 `base` 返回 `"/"`。
- `ext` 返回含 `.` 的扩展名，无扩展名返回 `""`。
