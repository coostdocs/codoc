---
weight: 11
title: "文件系统"
---


## 头文件

```cpp
#include "co/fs.h"
```

API 在 `fs` 命名空间，无需 `flag::parse`。


## 便捷函数

```cpp
bool  fs::exists(const char* path);
bool  fs::isdir (const char* path);
int64 fs::mtime (const char* path);  // 修改时间
int64 fs::fsize (const char* path);  // 文件大小

bool fs::mkdir(const char* path, bool p = false);
bool fs::mkdir(char* path, bool p);  // async-signal-safe

bool fs::rm(const char* path, bool r = false);
bool fs::mv(const char* from, const char* to);
bool fs::symlink(const char* dst, const char* lnk);
```

- 返回 `bool`，成功 `true`，失败 `false`。
- `mtime` / `fsize` 失败返回 `-1`。
- `mkdir`：`p=true` 等价 `mkdir -p`。
- `mkdir(char*, bool)` 为 async-signal-safe 版本，会临时修改并恢复 `path`。
- `rm`：`r=true` 递归删除；`path` 不存在返回 `true`；不跟随符号链接。
- `mv`：行为同 Linux `mv`。
- `symlink` 在 Windows 上需管理员权限。
- 出错后用 `co::error()` / `co::strerror()` 查询错误信息。
- 另有 `co::string` / `std::string` 版本重载。


## Windows 路径约定

- 用户传入 UTF-8 路径，coost 内部转 `wchar_t`。
- 返回给用户的路径也是 UTF-8。
- 用户无需关心 `wchar_t`，但要保证路径是合法 UTF-8。


## fs::file

### 打开模式

```text
'r': read         open if exists
'a': append       created if not exists
'w': write        created if not exists, truncated if exists
'm': modify       like 'w', but not truncated if exists
'+': read/write   created if not exists
```

### 定义

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

- 不可拷贝，只可移动。
- **内部无缓冲区**，Unix 基于 `read`/`write`，Windows 基于 `ReadFile`/`WriteFile`。
- `read(buf, n)` 返回读取字节数；到文件尾或出错返回 0。
- `seek` 返回 `bool`，`whence` 用 `seek_beg` / `seek_cur` / `seek_end`。
- 出错用 `co::error()` / `co::strerror()`。

### 示例

```cpp
fs::file f("test.txt", 'w');
if (!f) return;
f.write("hello ");
f.write("coost\n");
f.close();

// 重新以只读模式打开
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

- 不可拷贝，只可移动。
- `all()` 返回目录下所有条目名，不含 `.` 和 `..`。
- 迭代顺序由系统 API 决定，不保证顺序。
- `iterator::operator*` 返回条目名，不含路径。

### 示例

```cpp
fs::dir d(".");
for (auto&& name : d) {
    co::println("entry: ", name);
}
```

## 综合示例

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
