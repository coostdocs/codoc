---
weight: 4
title: "配置"
---

## 头文件

```cpp
#include <co/flag.h>
```

API 在 `flag` 命名空间。


## 概述

`flag` 提供命令行参数与配置文件解析功能，coost 中所有组件配置项都通过 flag 定义。

- 定义 flag 即定义全局变量，变量名是 `FLG_<name>`；
- 支持命令行参数与配置文件两种方式；
- 命令行中支持 `-help` 打印帮助信息、`-mkconf` 生成配置文件、`-version` 显示程序版本；


## 定义 flag

### DEF 宏

```cpp
DEF_bool(name, value, help, ...);
DEF_int32(name, value, help, ...);
DEF_int64(name, value, help, ...);
DEF_uint32(name, value, help, ...);
DEF_uint64(name, value, help, ...);
DEF_double(name, value, help, ...);
DEF_string(name, value, help, ...);
```

定义后通过全局变量访问，如 `FLG_name`。

类型对应关系：

| 宏 | 类型 | 内部标识 |
|---|---|---|
| `DEF_bool` | `bool` | `'b'` |
| `DEF_int32` | `int32` | `'i'` |
| `DEF_int64` | `int64` | `'I'` |
| `DEF_uint32` | `uint32` | `'u'` |
| `DEF_uint64` | `uint64` | `'U'` |
| `DEF_double` | `double` | `'d'` |
| `DEF_string` | `co::string&` | `'s'` |

示例：
```cpp
// 定义全局 bool 类型变量，变量名是 FLG_debug
DEF_bool(debug, false, "debug mode");
```

### DEC 宏

```cpp
DEC_bool(name);
DEC_int32(name);
DEC_int64(name);
DEC_uint32(name);
DEC_uint64(name);
DEC_double(name);
DEC_string(name);
```

用于跨文件声明，类型必须与 `DEF_xxx` 一致。

### 别名

flag 支持别名，命令行或配置文件中可以用别名。

`DEF_xxx` 最后一个变参可以指定最多一个别名：

```cpp
// d 是别名
DEF_bool(debug, false, "debug mode", d);
```

别名会显示在帮助信息中，例如 `-debug,d`。


## 解析参数

```cpp
co::vector<co::string> parse(int argc, char** argv, bool command_line_only=false);
```

- 解析命令行参数与配置文件，更新 flag 的值。
- 返回值包含所有非 flag 参数；
- `command_line_only == true` 时只解析命令行参数；
- 通常在 `main` 开头调用；
- 遇到错误时，打印信息，退出程序。


## 命令行参数

支持两种形式：

```bash
-name=value
-name value
```

`-` 可以是 1 个或多个，`-debug`、`--debug` 等价。

### bool 类型

```bash
-debug          # 等价于 -debug=true
-debug=true
-debug=false
```

### 整数单位

整数类型支持单位 `k, m, g, t, p`，不分大小写，`1k = 1024`。

```bash
-co_stack_size=2m   # 2 * 1024 * 1024
```

### 单字母 flag 简写语法

单字母 flag 支持如下两种简写(仅用于命令行)：

- 多个单字母 bool flag 可以合并：如 x、y、z 都是 bool flag，`-xyz` 可以将三者都设为 true。
- 单字母整型 flag 可以连写值：如 `-n8` 相当于 `-n=8`。


## 配置文件

默认命令行中第一个非 flag 参数(名字必须以 `.conf` 结尾)，作为配置文件：

```bash
./xx xx.conf
```

配置文件格式：

```ini
# 注释
debug = true
threads = 8
port = 8080
name = "my app"
n = 8k   # 8192 
```

- `#` 表示注释；
- 支持空行；
- 行首或行尾可以有空格；
- key 前面不加 `--`；
- `=` 前后可以有空格；
- 字符串首尾有空格时需要加引号，单引号和双引号都支持；
- 字符串支持常见转义字符，具体见 `co::string::unescape`；
- bool 支持 `true/false`、`1/0`，其余值都当作 `false`；
- 整数支持 `k, m, g, t, p` 单位，不区分大小写；
- 配置文件中出现未定义 flag 时，会在终端打印一行 warning 信息，不会退出程序；
- flag 名大小写敏感。


## 命令行与配置文件优先级

- 命令行参数与配置文件同时存在时，命令行参数会覆盖配置文件中的值；
- 命令行传入的 `.conf` 会覆盖 `set_config_path` 设置的默认路径。



## 内部 flag

flag 组件内部定义了三个 bool 类型 flag：

```cpp
DEF_bool(help, false, s_help);
DEF_bool(version, false, s_version);
DEF_bool(mkconf, false, s_mkconf);
```

### -help

打印帮助信息。

```bash
./xx -help
```

帮助信息格式示例：

```text
usage:  ./xx [xx.conf] [-flag [value]] [-flag=value]...

flags:  -name[,alias]  type  comments  (default value)
  -help       b  显示帮助信息  (false)
  -version,v  b  显示版本信息  (false)
  -mkconf     b  生成配置文件  (false)

  -boo        b  bool flag  (false)
  ...
```

若用户包含了 `co/log.h, co/co.h, co/rpc.h`，coost 内部定义的 flag 也会显示在帮助信息中。


### -version

显示程序版本，需要在 `flag::parse` 前调用 `flag::set_program_version` 设置版本号。未设置时版本信息为空。

```bash
./xx -version
```

### -mkconf

生成配置文件：

```bash
./xx -mkconf
```

- 在当前执行命令的目录生成配置文件；
- 文件名规则：可执行文件名去掉 `.exe`，再加上 `.conf`；
- 包含所有用户 flag，以及用到的 coost 组件中的 flag；
- 若不想 flag 出现在配置文件中，可以用 `flag::hide()` 隐藏 flag。


## 运行时 API

```cpp
// 添加别名，@new_name 必须有静态生命周期
// 如果 @new_name 为空，则移除已有别名
void flag::alias(const char* name, const char* new_name);

// 设置默认配置路径
void flag::set_config_path(const char* path);

// 设置程序版本
void flag::set_program_version(const char* ver);

// 隐藏 flag，使其不出现在帮助信息与 -mkconf 生成的配置文件中
void flag::hide(const char* name);

// 与 hide 相反
void flag::unhide(const char* name);

// 设置 flag 的值，出错时返回 false, 终端打印错误信息
bool flag::set_value(const char* flag_name, const char* value);

// 注册回调，在 flag::parse 解析完命令行参数后执行
void flag::run_after_parse(void(*cb)());

// 注册回调，在 flag::parse 解析命令行参数前执行
void flag::run_before_parse(void(*cb)());
```

{{< hint warning >}}
上述 API 均非线程安全，**需要在 `flag::parse` 前调用**。
{{< /hint >}}

- `alias` 添加最多一个别名。
- `set_config_path` 设置默认配置路径后，命令行中可以不传配置文件，`parse` 会从默认路径解析配置文件。
- `set_value` 中 `value` 是字符串形式，内部按 flag 类型解析，出错不会退出程序；

示例:

```cpp
flag::alias("version", "v");
flag::set_value("debug", "true");
```

## 线程安全

flag 本质是全局变量或对象，若有多个线程访问或修改 `FLG_xxx`。用户需要自己保证并发安全。



## 示例

```cpp
#include <co/log.h> // 已包含 co/flag.h

DEF_bool(debug, false, "enable debug mode");
DEF_int32(threads, 4, "number of threads");
DEF_uint32(port, 8080, "server port");
DEF_string(name, "coost", "app name", n); // n 为别名

int main(int argc, char** argv) {
    // 设置程序版本号
    flag::set_program_version("1.0.0");

    // 日志组件内置 flag，日志也输出到终端
    flag::set_value("also_log2console", "true");
    
    // 可选：设置默认配置文件路径
    // flag::set_config_path("my.conf");

    auto non_flags = flag::parse(argc, argv);

    log::info("debug=", FLG_debug);
    log::info("threads=", FLG_threads);
    log::info("port=", FLG_port);
    log::info("name=", FLG_name);

    for (auto& s : non_flags) {
        log::info("non-flag: ", s);
    }

    return 0;
}
```

运行：

```bash
./app
./app -debug -threads=8 -port 8080 -name=myapp
./app -help
./app -version
./app -mkconf
./app app.conf
```
