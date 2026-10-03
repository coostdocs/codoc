---
weight: 11
title: "File System"
---

## Header

```cpp
#include "co/fs.h"
```

The API is in the `fs` namespace; `flag::parse` is not required.

## Convenience Functions

```cpp
bool  fs::exists(const char* path);
bool  fs::isdir (const char* path);
int64 fs::mtime (const char* path);  // modification time
int64 fs::fsize (const char* path);  // file size

bool fs::mkdir(const char* path, bool p = false);
bool fs::mkdir(char* path, bool p);  // async-signal-safe

bool fs::rm(const char* path, bool r = false);
bool fs::mv(const char* from, const char* to);
bool fs::symlink(const char* dst, const char* lnk);
```

- Returns `bool`: `true` on success, `false` on failure.
- `mtime` / `fsize` return `-1` on failure.
- `mkdir`: `p=true` is equivalent to `mkdir -p`.
- `mkdir(char*, bool)` is the async-signal-safe version, which temporarily modifies and then restores `path`.
- `rm`: `r=true` deletes recursively; returns `true` if `path` does not exist; does not follow symbolic links.
- `mv`: behaves like Linux `mv`.
- `symlink` requires administrator privileges on Windows.
- After an error, use `co::error()` / `co::strerror()` to query error information.
- There are also overloads for `co::string` / `std::string`.

## Windows Path Convention

- Users pass in UTF-8 paths; coost internally converts them to `wchar_t`.
- Paths returned to users are also UTF-8.
- Users do not need to care about `wchar_t`, but must ensure that paths are valid UTF-8.

## fs::file

### Open Modes

```text
'r': read         open if exists
'a': append       created if not exists
'w': write        created if not exists, truncated if exists
'm': modify       like 'w', but not truncated if exists
'+': read/write   created if not exists
```

### Definition

```cpp
struct file {
    enum _seekfrom_t { seek_beg = 0, seek_cur = 1, seek_end = 2 };

    file();
    explicit file(size_t n);
    file(const char* path, char mode);
    file(const co::string& path, char mode);
    file(const std::string& path, char mode);
    file(file&& f) noexcept;
    ~file();

    file(const file&) = delete;
    void operator=(const file&) = delete;
    void operator=(file&&) = delete;

    explicit operator bool() const noexcept;
    const char* path() const noexcept;
    int64 size() const noexcept;
    bool  exists() const noexcept;

    bool open(const char* path, char mode);
    bool open(const co::string& path, char mode);
    bool open(const std::string& path, char mode);
    void close();

    bool seek(int64 off, _seekfrom_t whence = seek_beg);
    size_t read(void* buf, size_t n);
    co::string read(size_t n);
    size_t write(const void* s, size_t n);
    size_t write(const char* s);
    size_t write(const co::string& s);
    size_t write(const std::string& s);
    size_t write(char c);

    void* _p;
};
```

- Non-copyable, movable only.
- **No internal buffer**; on Unix it is based on `read`/`write`, and on Windows it is based on `ReadFile`/`WriteFile`.
- `read(buf, n)` returns the number of bytes read; returns 0 at end of file or on error.
- `seek` returns `bool`; `whence` uses `seek_beg` / `seek_cur` / `seek_end`.
- On error, use `co::error()` / `co::strerror()`.

### Example

```cpp
fs::file f("test.txt", 'w');
if (!f) return;
f.write("hello ");
f.write("coost\n");
f.close();

// Reopen in read-only mode
f.open("test.txt", 'r');
co::string s = f.read(32);
co::println("content = ", s);
```

## fs::dir

```cpp
struct dir {
    dir();
    explicit dir(const char* path);
    explicit dir(const co::string& path);
    explicit dir(const std::string& path);
    dir(dir&& d) noexcept;
    ~dir();

    dir(const dir&) = delete;
    void operator=(const dir&) = delete;
    void operator=(dir&&) = delete;

    bool open(const char* path);
    bool open(const co::string& path);
    bool open(const std::string& path);
    void close();
    const char* path() const noexcept;

    co::vector<co::string> all() const;

    struct iterator {
        co::string operator*() const;
        iterator& operator++();
        bool operator==(const iterator& it) const noexcept;
        bool operator!=(const iterator& it) const noexcept;
    };

    iterator begin() const;
    iterator end() const;
};
```

- Non-copyable, movable only.
- `all()` returns all entry names in the directory, excluding `.` and `..`.
- The iteration order is determined by the system API and is not guaranteed.
- `iterator::operator*` returns the entry name, without the path.

### Example

```cpp
fs::dir d(".");
for (auto&& name : d) {
    co::println("entry: ", name);
}
```

## Comprehensive Example

```cpp
#include "co/fs.h"
#include "co/print.h"

int main() {
    co::println("exists(.) = ", fs::exists("."));
    co::println("isdir(.)  = ", fs::isdir("."));

    fs::mkdir("tmp_fs_test/sub", true);

    fs::file f("tmp_fs_test/a.txt", 'w');
    f.write("hello coost\n");
    f.close();

    fs::file r("tmp_fs_test/a.txt", 'r');
    co::println("content = ", r.read(1024));

    fs::dir d("tmp_fs_test");
    for (auto& name : d) co::println("entry: ", name);

    fs::mv("tmp_fs_test/a.txt", "tmp_fs_test/b.txt");
    fs::rm("tmp_fs_test", true);
    return 0;
}
```
