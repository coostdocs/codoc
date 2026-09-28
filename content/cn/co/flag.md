---
weight: 4
title: "配置"
---

## 头文件

```cpp
#include <co/flag.h>
```

如果同时使用日志，可以只包含 `co/log.h`，它已包含 `co/flag.h`。

API 在 `flag` 命名空间。

---

## 概述

`flag` 提供命令行参数与配置文件解析功能，coost 中所有组件的配置项都通过 flag 定义。

- 定义 flag 即定义全局变量，变量名是 `FLG_<name>`；
- 支持命令行参数与配置文件两种方式；
- 命令行中支持 `-help` 打印帮助信息、`-mkconf` 生成配置文件、`-version` 显示程序版本；

---

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

定义后通过全局变量访问，例如 `FLG_name`。

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

用于跨文件声明，类型必须与 `DEF_xxx` 一致，否则同一项目中编译会报错。


### string 类型

`DEF_string` 中 `FLG_<name>` 是 `co::string&`，内部由 coost 内存分配器管理。默认值可以是 `const char*`、`co::string`、`std::string` 等。

---

## 解析参数

```cpp
co::vector<co::string> parse(int argc, char** argv, bool command_line_only=false);
```

- 返回值包含所有非 flag 参数；
- 默认同时解析命令行参数和配置文件；
- `command_line_only == true` 时只解析命令行参数；
- 通常在 `main` 开头调用；
- 遇到错误时，打印错误信息，并退出程序。

---

## 命令行参数

支持两种形式：

```bash
-name=value
-name value
```

`-` 的数量可以是 1 个或多个，`-debug`、`--debug`、`---debug` 等价。

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

### 单字母 flag 的简写语法

coost 对单字母 flag 提供了两种简写，仅用于命令行，配置文件不支持：

- 多个单字母 bool flag 可以合并：假设 x、y、z 都是 bool flag，则 `-xyz` 可以将三者都置为 true。
- 单字母整数 flag 可以连写值：`-n8` 相当于 `-n=8`。

### 别名

`DEF_xxx` 的最后一个变参可以给 flag 指定最多一个别名：

```cpp
DEF_string(version, "3.0", "xxx", v); // v 是 version 的别名
```

别名会显示在帮助信息中，例如 `-version,v`；配置文件中也可以使用别名。

---

## 配置文件

默认第一个非 flag 参数且以 `.conf` 结尾的文件作为配置文件：

```bash
./xx xx.conf
```

多个 `.conf` 参数时，只有第一个作为配置文件。

也可以用 `set_config_path` (需要在 parse 前调用)设置默认配置文件路径，命令行传入的 `.conf` 会覆盖它。

配置文件格式：

```ini
# 注释
debug = true
threads = 8
port = 8080
name = "my app"
n = 8M
```

规则：

- `#` 表示注释；
- 支持空行；
- 行首或行尾可以有空格；
- key 不加 `--`；
- `=` 前后可以有空格；
- string 首尾有空格时需要加引号，单引号和双引号都支持；
- 字符串中支持常见转义字符，具体见 `co::string::unescape`；
- bool 支持 `true/false`、`1/0`，其余值都当作 `false`；
- 整数支持 `k, m, g, t, p` 单位；
- 配置文件中可以使用别名；
- 配置文件中出现未定义 flag 时，会在终端打印一行 warning 信息，不会退出程序；
- flag 名大小写敏感。

### 优先级

- 命令行参数与配置文件同时存在时，命令行参数会覆盖配置文件中的值；
- 命令行传入的 `.conf` 覆盖 `set_config_path` 设置的默认路径。

---

## 内部 flag

coost 内部定义了三个 bool 类型 flag：

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
usage:  flag [flag.conf] [-flag [value]] [-flag=value]...

flags:  -name[,alias]  type  comments  (default value)
  -help       b  显示帮助信息  (false)
  -version,v  b  显示版本信息  (false)
  -mkconf     b  生成配置文件  (false)

  -boo        b  bool flag  (false)
  ...
```

如果用户 include 了 coost 相关组件头文件，coost 组件内部定义的 flag 会显示在帮助信息中。

### -version

显示程序版本，需要在 `flag::parse` 前调用 `flag::set_program_version` 设置版本号。未设置时打印空字符串。

### -mkconf

生成配置文件：

```bash
./xx -mkconf
```

- 在当前执行命令的目录生成配置文件；
- 文件名规则：可执行文件名去掉 `.exe`，再加上 `.conf`；
- 不包含 `help`、`version`、`mkconf`；
- 包含所有用户 flag，除非被 `flag::hide` 隐藏；
- 如果用户 include 了 coost 相关组件，组件内部 flag 也会出现在生成的配置文件中。

---

## 运行时 API

```cpp
// 添加别名，@new_name 必须有静态生命周期
// 如果 @new_name 为空，则移除已有别名
void flag::alias(const char* name, const char* new_name);

// 设置默认配置文件路径
void flag::set_config_path(const char* path);

// 设置程序版本，需在 flag::parse 前调用
void flag::set_program_version(const char* ver);

// 隐藏 flag，使其不出现在帮助信息与 -mkconf 生成的配置文件中
void flag::hide(const char* name);

// 与 hide 相反
void flag::unhide(const char* name);

// 设置 flag 的值，出错时返回 false
bool flag::set_value(const char* flag_name, const char* value);

// 注册回调，在 flag::parse 解析完命令行参数后执行
void flag::run_after_parse(void(*cb)());

// 注册回调，在 flag::parse 解析命令行参数前执行
void flag::run_before_parse(void(*cb)());
```

### alias

- `new_name` 必须有静态生命周期；
- 如果 `new_name` 为空，则移除已有别名；
- 别名会显示在帮助信息中，例如 `-version,v`；
- 配置文件中可以使用别名；
- 必须在 `flag::parse` 前调用，否则 `flag::parse` 看不到这个别名，`-help` 也不会显示。

### set_config_path

- 设置默认配置文件路径，默认值为空；
- 设置后，用户在命令行中可以不传配置文件路径，`flag` 会从默认路径解析配置文件；
- 需在 `flag::parse` 前调用；
- 多次调用时，后设置的值覆盖之前的值；
- 命令行传入的 `.conf` 会覆盖这个默认路径。

### set_program_version

- 设置程序版本号；
- 必须在 `flag::parse` 前调用，若在 `flag::parse` 后调用，`./xx -version` 无法显示版本信息。

### hide / unhide

- `hide` 隐藏 flag，使其不出现在 `-help` 和 `-mkconf` 生成的配置文件中；
- `unhide` 与 `hide` 相反；
- 必须在 `flag::parse` 前调用，因为 `-help` / `-mkconf` 的输出是 `flag::parse` 解析后立即执行的。

### set_value

- 按名字设置 flag 的值，`value` 是字符串形式，内部按 flag 类型解析；
- 出错时返回 `false`，并用 `co::println` 打印错误信息，不会退出程序；
- 一般在 `flag::parse` 前调用，用于修改 flag 默认值，这样 `flag::parse` 传入的值依旧可以覆盖它；
- 在 `flag::parse` 后调用，会覆盖掉命令行或配置文件中确定的值，使命令行、配置文件中的设置失去作用，通常不建议这样做。

### run_after_parse

- 注册的回调在 `flag::parse` 解析完命令行参数与配置文件后执行；
- 允许注册多个回调，按注册顺序执行；
- **coost 内部用它启动协程调度线程、日志线程**，所以必须调用 `flag::parse`，这些线程才会启动。

### run_before_parse

- 注册的回调在 `flag::parse` 解析参数前执行；
- 允许注册多个回调，按注册顺序执行；
- coost 内部用它来 unhide 组件相关的 flag，例如包含 `co/rpc.h` 后，RPC 组件会在回调中调用 `flag::unhide("rpc_max_msg_size")` 等。

---

## 运行时 API 的调用时机

`flag::parse` 是解析的入口。`-help`、`-version`、`-mkconf` 这三个内部 flag，都是 `flag::parse` 解析到后立即执行并退出的。因此：

> 所有影响 `-help` / `-version` / `-mkconf` 输出内容、影响 `flag::parse` 解析行为的配置，都必须在 `flag::parse` 前设置。

具体来说：

- `set_program_version`：必须在 parse 前，否则 `-version` 读不到版本号；
- `alias`：必须在 parse 前，否则 `-help` 看不到别名，命令行也不能用别名；
- `set_config_path`：必须在 parse 前，因为配置文件读取发生在 parse 过程中；
- `hide` / `unhide`：必须在 parse 前，因为 `-help` / `-mkconf` 的输出在 parse 中生成；
- `run_before_parse` / `run_after_parse`：需要在 parse 前注册才有效。

`set_value` 是个例外：

- parse 前调用：修改默认值，parse 时命令行 / 配置文件仍可覆盖；
- parse 后调用：直接覆盖当前值，使命令行 / 配置文件中的设置失去作用，一般不建议这样用。


---

## 线程安全

flag 本质是全局变量或全局对象，访问 `FLG_xxx` 不是线程安全的。需要用户自己保证并发安全。

---

## 示例

```cpp
#include <co/log.h> // 已包含 co/flag.h

DEF_bool(debug, false, "enable debug mode");
DEF_int32(threads, 4, "number of threads");
DEF_uint32(port, 8080, "server port");
DEF_string(name, "coost", "app name", n); // n 为别名

int main(int argc, char** argv) {
    flag::set_program_version("1.0.0");
    flag::set_value("also_log2console", "true");
    
    // 可选：设置默认配置文件路径
    // flag::set_config_path("my.conf");

    co::vector<co::string> non_flags = flag::parse(argc, argv);

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
./app -debug -threads=8 -port 8080 -name=myapp xx.conf
```

配置文件 `xx.conf`：

```ini
# comment
debug = true
threads = 8
port = 8080
name = "my app"
```

生成配置：

```bash
./app -mkconf
```

查看帮助：

```bash
./app -help
```

查看版本：

```bash
./app -version
```

