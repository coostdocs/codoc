---
weight: 12
title: "操作系统"
---


## 头文件

```cpp
#include "co/os.h"
```

API 在命名空间 `os`，无需 `flag::parse`。


## 环境变量

```cpp
co::string os::env(const char* name);
bool       os::env(const char* name, const char* value);
```

- `env(name)`：获取环境变量值，不存在返回空字符串。
- `env(name, value)`：设置环境变量值，`value` 为 `nullptr` 或空字符串时删除该变量，成功返回 `true`。
- 值不做 UTF-8 转码。


## 路径

```cpp
co::string os::homedir();  // 当前用户 home 目录
co::string os::cwd();      // 当前工作目录
co::string os::exepath();  // 可执行文件路径
co::string os::exedir();   // 可执行文件所在目录
co::string os::exename();  // 可执行文件名
```

- 失败返回空字符串。
- Windows 上返回 UTF-8，与 `fs` 一致。
- `exename()` 含扩展名。


## 进程与硬件

```cpp
int    os::pid();
int    os::cpunum();
size_t os::pagesize();
```

- `pid()`：当前进程 id。
- `cpunum()`：逻辑 CPU 核数。
- `pagesize()`：页大小（字节）。


## 信号

```cpp
typedef void (*sig_handler_t)(int);

sig_handler_t os::signal(int sig, sig_handler_t handler, int flag=0);
```

- 只是 `::signal` 的简单包装。
- 返回旧的处理函数。
- Windows 上不支持 `flag`。


## 执行命令

```cpp
bool os::system(const char* cmd);
```

- 无 `std::string` 重载，需先 `.c_str()`。
- 非 Windows 基于 `popen` / `pclose`，输出直接到终端。
- 返回 `bool`，表示命令是否执行成功。


## 示例

```cpp
#include "co/os.h"
#include "co/print.h"

int main() {
    co::println("homedir = ", os::homedir());
    co::println("cwd     = ", os::cwd());
    co::println("exepath = ", os::exepath());
    co::println("pid     = ", os::pid());
    co::println("cpunum  = ", os::cpunum());
    co::println("page    = ", os::pagesize());

    co::println("PATH = ", os::env("PATH"));

    os::env("MY_VAR", "hello");
    co::println("MY_VAR = ", os::env("MY_VAR"));

    os::env("MY_VAR", nullptr);

    os::system("echo hello coost");
    return 0;
}
```
