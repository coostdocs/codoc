---
weight: 5
title: "Logging"
---

## Header

```cpp
#include <co/log.h>
```

The API is in the `log` namespace.

## Initialization and Shutdown

`log` internally uses the `flag` component to define some configuration items, and logging behavior can be controlled through command-line arguments or a configuration file. In addition, the logging thread starts only after command-line arguments are parsed. Therefore, it must be called at the beginning of the `main` function:

```cpp
flag::parse(argc, argv);
```

Shutdown API:
```cpp
void log::close();
```

- Flushes the log buffer and exits the logging thread.
- Users generally do not need to call this function explicitly.

## Printing Logs

Use the following functions to print logs at different levels:

```cpp
log::debug(...);
log::info(...);
log::warn(...);
log::error(...);
log::fatal(...);
```

- The above functions are all thread-safe and accept any number of arguments;
- Arguments can be any type supported by `co::string::operator<<`;
- A newline is automatically added at the end of the log;
- `fatal` level logs terminate the program.

Example:

```cpp
log::info("hello ", false, ' ', 23);
```

Supported argument types include:
- bool, outputs false or true;
- Character types: `char, signed char, unsigned char`;
- Integer types;
- Floating-point types: `double, float`;
- String types: `const char*`, `co::string`, `std::string`, `std::string_view`;
- Pointer types: `void*, T*`, outputs `0x` followed by the hexadecimal value;
- STL containers: `std::vector, std::map, co::vector, co::map`, etc.; `co/stl.h` must be included.

Custom types need to implement `operator<<(co::string&, const T&)`, for example:

```cpp
struct Point {
    int x, y;
};

inline co::string& operator<<(co::string& s, const Point& p) {
    s << "Point(" << p.x << ", " << p.y << ")";
    return s;
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);

    Point p{1, 2};
    co::println("point: ", p);
    log::info("point: ", p);

    return 0;
}
```

## Assertions

```cpp
log::check(cond, ...);
log::check_eq(a, b, ...);
log::check_ne(a, b, ...);
log::check_lt(a, b, ...);
log::check_gt(a, b, ...);
log::check_le(a, b, ...);
log::check_ge(a, b, ...);
```

- If check fails, it flushes the log buffer, prints stack information, and calls `abort()` to terminate the program;
- If check succeeds, no information is output.

Example:

```cpp
log::check(1 + 1 == 2, "1+1 should be 2");
log::check_eq(1 + 1, 2, "1+1 must == 2");
log::check_ne(1 + 1, 3, "1+1 != 3");
log::check_lt(1, 2, "1 < 2");
log::check_gt(2, 1, "2 > 1");
log::check_le(2, 2, "2 <= 2");
log::check_ge(2, 2, "2 >= 2");
```

## Printing Stack Information

coost captures exceptions or abnormal signals generated in the program and prints stack information before the program crashes and exits.

On non-Windows platforms, [libbacktrace](https://github.com/ianlancetaylor/libbacktrace) is required. On Linux, GCC generally has a built-in backtrace library; on macOS, users usually need to install it manually.

On non-Windows platforms, enable this feature during build as follows:

```bash
# xmake
xmake f --with_backtrace=true   # Use the backtrace library
xmake                           # Build libco
xmake b stack                   # Build test/stack.cc
xmake r stack                   # Run the stack test program

# cmake
mkdir cmakebuild && cd cmakebuild
cmake .. -DWITH_BACKTRACE=ON
make -j8
```

## Write Log Callback

By default, coost logs to a local file. Users can set a callback to write logs:

```cpp
void log::set_write_cb(
    void(*cb)(const void* data, size_t size),
    bool also_log2local=false
);
```

- `data` may contain multiple logs, and `size` may be large; be careful when using `UDP` to send logs.
- The callback is executed in the logging thread (**only a single thread writes logs**).
- If `also_log2local == true`, logs are also written to a local file.

{{< hint warning >}}
This needs to be called before `flag::parse`. After `flag::parse` parses command-line arguments, the logging thread starts; it is safer to set the callback before the logging thread starts.
{{< /hint >}}

Example:

```cpp
#include <co/log.h>
#include <stdio.h>

static void my_write_cb(const void* data, size_t size) {
    fwrite(data, 1, size, stderr);
}

int main(int argc, char** argv) {
    log::set_write_cb(my_write_cb, false);
    flag::parse(argc, argv);

    log::info("hello ", 23);
    log::warn("warn ", false);
    return 0;
}
```

## Log Configuration Items

| flag name             |     Type |                 Default | Meaning                                      |
| --------------------- | -------: | ----------------------: | -------------------------------------------- |
| `log_dir`             |   string |               `"logs"` | Log directory                                |
| `min_log_level`       |   uint32 |                    `0` | Minimum level of logs to output; 0-4 correspond to debug/info/warn/error/fatal |
| `max_log_size`        |   uint32 |                 `4096` | Maximum size of a single log                 |
| `max_log_file_size`   |    int64 | `256 << 20`, i.e. 256MB | Maximum log file size                        |
| `max_log_file_num`    |   uint32 |                    `8` | Maximum number of log files                  |
| `max_log_buffer_size` |   uint32 |   `32 << 20`, i.e. 32MB | Maximum log buffer size                      |
| `log_flush_ms`        |   uint32 |                  `128` | Time interval for flushing the log buffer, in milliseconds |
| `also_log2console`    |     bool |                `false` | Also output logs to the terminal             |
| `log_daily`           |     bool |                `false` | Rotate log files daily                       |

`min_log_level` can filter out lower-level logs; only logs with a level greater than or equal to `min_log_level` are output.

## Minimal Example

```cpp
#include <co/log.h>

int main(int argc, char** argv) {
    # parse is required; after parsing command-line arguments, the logging thread starts
    flag::parse(argc, argv);

    log::debug("This is debug.. ", 23);
    log::info("This is info.. ", 23);
    log::warn("This is warn.. ", 23);
    log::error("This is error.. ", 23);

    log::check(1 + 1 == 2, "1+1 should be 2");
    log::check_eq(1 + 1, 2, "1+1 must == 2");
    log::check_ne(1 + 1, 3, "1+1 != 3");
    log::check_lt(1, 2, "1 < 2");
    log::check_gt(2, 1, "2 > 1");
    log::check_le(2, 2, "2 <= 2");
    log::check_ge(2, 2, "2 >= 2");

    // log::fatal("fatal.. ", 23); // Will terminate the program

    return 0;
}
```

Run example:

```bash
./app -min_log_level=1 -also_log2console=true
```

The command-line arguments indicate that only info and higher-level logs are output, and they are output to the terminal.
