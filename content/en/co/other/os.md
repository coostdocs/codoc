---
weight: 12
title: "Operating System"
---

## Header

```cpp
#include "co/os.h"
```

The API is in the `os` namespace; `flag::parse` is not required.

## Environment Variables

```cpp
co::string os::env(const char* name);
bool       os::env(const char* name, const char* value);
```

- `env(name)`: gets the value of an environment variable; returns an empty string if it does not exist.
- `env(name, value)`: sets the value of an environment variable. If `value` is `nullptr` or an empty string, the variable is deleted. Returns `true` on success.
- The value is not UTF-8 transcoded.

## Paths

```cpp
co::string os::homedir();  // current user's home directory
co::string os::cwd();      // current working directory
co::string os::exepath();  // executable file path
co::string os::exedir();   // directory containing the executable
co::string os::exename();  // executable file name
```

- Returns an empty string on failure.
- On Windows, returns UTF-8, consistent with `fs`.
- `exename()` includes the extension.

## Process and Hardware

```cpp
int    os::pid();
int    os::cpunum();
size_t os::pagesize();
```

- `pid()`: current process id.
- `cpunum()`: number of logical CPU cores.
- `pagesize()`: page size (in bytes).

## Signals

```cpp
typedef void (*sig_handler_t)(int);

sig_handler_t os::signal(int sig, sig_handler_t handler, int flag=0);
```

- Just a simple wrapper around `::signal`.
- Returns the old handler.
- `flag` is not supported on Windows.

## Executing Commands

```cpp
bool os::system(const char* cmd);
```

- There is no `std::string` overload; you need to call `.c_str()` first.
- On non-Windows, it is based on `popen` / `pclose`; output goes directly to the terminal.
- Returns `bool`, indicating whether the command executed successfully.

## Example

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
