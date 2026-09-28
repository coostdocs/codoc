---
weight: 5
title: "日志"
---


## 头文件

```cpp
#include <co/log.h>
```

`co/log.h` 已经包含 `co/flag.h`。

API 在 `log` 命名空间。

---

## 初始化与关闭

`log` 内部使用 `flag` 组件定义了一些配置项，可通过命令行参数或配置文件控制日志行为。另外，日志线程在命令行参数解析后才会启动。因此必须在 `main` 函数开头调用：

```cpp
flag::parse(argc, argv);
```

关闭 API：
```cpp
void log::close();
```

说明：
- 用户一般不需要显示调用此 API。
- 调用此函数，会刷新日志缓冲，退出日志线程。

---

## 打印日志

使用如下函数打印不同级别的日志：

```cpp
log::debug(...);
log::info(...);
log::warn(...);
log::error(...);
log::fatal(...);
```

说明：
- 日志函数是线程安全的；
- 日志函数接受任意数量的参数；
- 参数可以是 `co::string::operator<<` 支持的任意类型；
- 日志函数自动在尾部添加换行符；
- `fatal` 级别的日志会终止程序运行。

示例：
```cpp
log::info("hello ", false, ' ', 23);
```

支持的参数类型有：
- bool，输出 false 或 true；
- 字符类型：`char, signed char, unsigned char`；
- 整数类型；
- 浮点数类型：`double, float`；
- 字符类型：`char, signed char, unsigned char`；
- 字符串类型：`const char*`, `std::string`, `co::string`；
- 指针类型：`void*, T*`，输出 `0x` 开头的十六进制值；
- STL 容器类型：`std::vector, std::map, co::vector, co::map` 等，需要包含 `co/stl.h`。

自定义类型需要实现：
```cpp
operator<<(co::string&, const T&);
```

示例：

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
    log::info("point: ", p);

    return 0;
}
```

---

## 断言

提供以下断言接口：
```cpp
log::check(cond, ...);
log::check_eq(a, b, ...);
log::check_ne(a, b, ...);
log::check_lt(a, b, ...);
log::check_gt(a, b, ...);
log::check_le(a, b, ...);
log::check_ge(a, b, ...);
```

示例：
```cpp
log::check(1 + 1 == 2, "1+1 should be 2");
log::check_eq(1 + 1, 2, "1+1 must == 2");
log::check_ne(1 + 1, 3, "1+1 != 3");
log::check_lt(1, 2, "1 < 2");
log::check_gt(2, 1, "2 > 1");
log::check_le(2, 2, "2 <= 2");
log::check_ge(2, 2, "2 >= 2");
```

check 失败行为：
- 将缓存中日志输出到目标。
- 打印堆栈信息。
- 调用 `abort()` 终止程序。

---

## 写日志回调

coost 日志默认写本地文件，用户可以调用如下 API 设置一个 callback，自定义日志输出目标：
```cpp
void log::set_write_cb(
    void(*cb)(const void* data, size_t size),
    bool also_log2local=false
);
```

说明：
- `data` 中可能包含多条日志，`size` 可能很大，用 `UDP` 发送日志时需要注意。
- callback 在写日志的线程中执行(**coost只有单个线程写日志**)。
- 如果 `also_log2local == true`，日志也写本地文件。
- 该函数一般需要在 `flag::parse` 前调用；

示例：

```cpp
#include <co/log.h>
#include <cstdio>

static void my_write_cb(const void* data, size_t size) {
    fwrite(data, 1, size, stdout);
}

int main(int argc, char** argv) {
    log::set_write_cb(my_write_cb, false);
    flag::parse(argc, argv);

    log::info("hello ", 23);
    log::warn("warn ", false);

    return 0;
}
```

---

## 日志配置项

`log` 内部通过 [flag](../flag/) 定义了一些配置项，可通过命令行参数与配置文件控制日志行为。

| flag 名                |     类型 |                 默认值 | 含义                                           |
| --------------------- | -----: | ------------------: | -------------------------------------------- |
| `log_dir`             | string |            `"logs"` | 日志目录                                         |
| `min_log_level`       | uint32 |                 `0` | 输出日志的最小级别，0-4 对应 debug/info/warn/error/fatal |
| `max_log_size`        | uint32 |              `4096` | 单条日志最大大小                                     |
| `max_log_file_size`   |  int64 | `256 << 20`，即 256MB | 日志文件最大大小                                     |
| `max_log_file_num`    | uint32 |                 `8` | 日志文件最大数量                                     |
| `max_log_buffer_size` | uint32 |   `32 << 20`，即 32MB | 日志缓存最大大小                                     |
| `log_flush_ms`        | uint32 |               `128` | 刷新日志缓存的时间间隔，单位毫秒                             |
| `also_log2console`    |   bool |             `false` | 日志也输出到终端                                     |
| `log_daily`           |   bool |             `false` | 日志文件按天轮转                                     |

`min_log_level` 语义：

- `0`：debug
- `1`：info
- `2`：warn
- `3`：error
- `4`：fatal

日志级别大于等于 `min_log_level` 才输出。

`min_log_level` 过滤在产生日志的线程完成。

---

## 最小示例

```cpp
#include <co/log.h>

int main(int argc, char** argv) {
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

    // log::fatal("fatal.. ", 23); // 会终止程序

    return 0;
}
```

运行示例：

```bash
./app -min_log_level=1 -also_log2console=true
```

命令行参数表示只输出 info 及以上级别日志，并输出到终端。
