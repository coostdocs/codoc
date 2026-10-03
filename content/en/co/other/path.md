---
weight: 10
title: "File Path"
---

## Header

```cpp
#include "co/path.h"
```

The API is in the `path` namespace, not `co`.

Ported from Go's `path` package; the path separator is fixed as `/`.

## Overview

- `clean`: returns the shortest equivalent path.
- `join`: joins multiple path elements.
- `split`: splits into dir and file at the last `/`.
- `dir`: returns the directory part.
- `base`: returns the last segment.
- `ext`: returns the extension.

All functions returning `co::string` have three overloads: `(const char*, size_t)`, `(const char*)`, and `(const co::string&)`.

## clean

```cpp
co::string path::clean(const char* s, size_t n);
```

Returns the shortest equivalent path.

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

Joins any number of path elements; the result is `clean`ed; empty elements are ignored.

```cpp
path::join("", "")       // -> ""
path::join("/x", "y")    // -> "/x/y"
path::join("/x/", "y")   // -> "/x/y"
```

## split

```cpp
std::pair<co::string, co::string> path::split(const char* s, size_t n);
```

Splits into dir and file at the last `/`, satisfying `path = dir + file`.

```cpp
path::split("/a/")   // -> <"/a/", "">
path::split("/a/b")  // -> <"/a/", "b">
```

## dir

```cpp
co::string path::dir(const char* s, size_t n);
```

Returns the directory part; the result is `clean`ed.

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

Returns the last segment, after removing trailing slashes.

```cpp
path::base("")       // -> "."
path::base("/a/b")   // -> "b"
path::base("/a/b/")  // -> "b"
```

If the path consists entirely of slashes, returns `"/"`.

## ext

```cpp
co::string path::ext(const char* s, size_t n);
```

Returns the file extension (including `.`).

```cpp
path::ext("x/x.c")  // -> ".c"
path::ext("a/b")    // -> ""
path::ext("/b.c/")  // -> ""
path::ext("a.")     // -> "."
```

## Example

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

## Notes

- The path separator is fixed as `/` and is platform-independent.
- `clean` is the basis of the other functions; all results are cleaned.
- `join` ignores empty elements.
- The result of `split` satisfies `path = dir + file`.
- `base("")` and `dir("")` return `"."`.
- If the path consists entirely of slashes, `base` returns `"/"`.
- `ext` returns the extension including `.`, and returns `""` if there is no extension.
