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


## 概述

`flag` 提供命令行参数与配置文件解析功能，coost 中所有组件的配置项都通过 flag 定义。

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

示例：
```cpp
// 定义全局 bool 类型变量，变量名是 FLG_debug
DEF_bool(debug, false, "enable debug mode");
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

用于跨文件声明，类型必须与 `DEF_xxx` 一致，否则同一项目中编译会报错。

### 别名

flag 支持别名，命令行或配置文件中可以用别名替代原名。

`DEF_xxx` 的最后一个变参可以给 flag 指定最多一个别名：

```cpp
// d 是 debug 的别名
DEF_bool(debug, false, "debug mode", d);
```

别名会显示在帮助信息中，例如 `-version,v`。


## 解析参数

```cpp
co::vector<co::string> parse(int argc, char** argv, bool command_line_only=false);
```

- 解析命令行参数与配置文件，更新 flag 的值。
- 返回值包含所有非 flag 参数；
- `command_line_only == true` 时只解析命令行参数；
- 通常在 `main` 开头调用；
- 遇到错误时，打印错误信息，并退出程序。


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

- 多个单字母 bool flag 可以合并：如 x、y、z 都是 bool flag，则 `-xyz` 可以将三者都置为 true。
- 单字母整数类型 flag 可以连写值：如 `-n8` 相当于 `-n=8`。


## 配置文件

默认命令行中第一个非 flag 参数且名字以 `.conf` 结尾的文件作为配置文件：

```bash
./xx xx.conf
```

有多个 `.conf` 参数时，只有第一个作为配置文件。

也可以用 `set_config_path` (需要在 parse 前调用)设置默认配置文件路径，命令行传入的 `.conf` 会覆盖它。

配置文件格式：

```ini
# 注释
debug = true
threads = 8
port = 8080
name = "my app"
n = 8k   # 8192 
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

如果用户包含了 coost 相关组件头文件(co/log.h, co/co.h, co/rpc.h)，coost 内部定义的 flag 会显示在帮助信息中。


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
- 包含所有用户 flag，以及用到的 coost 内部组件中的 flag；
- 如果不想 flag 出现在配置文件中，可以用 `flag::hide()` 隐藏 flag。



## 运行时 API

```cpp
// 添加别名，@new_name 必须有静态生命周期
// 如果 @new_name 为空，则移除已有别名
void flag::alias(const char* name, const char* new_name);

// 设置默认配置文件路径
void flag::set_config_path(const char* path);

// 设置程序版本
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

- 添加别名，最多只能有一个别名；
- `new_name` 必须有静态生命周期；
- 如果 `new_name` 为空，则移除已有别名；
- 别名会显示在帮助信息中，例如 `-version,v`；
- **必须在 `flag::parse` 前调用**，否则 `flag::parse` 看不到这个别名，`-help` 也不会显示。


### set_config_path

- 设置默认配置文件路径；
- 设置后，用户在命令行中可以不传配置文件参数，`flag::parse` 会从默认路径解析配置文件；
- 多次调用时，会覆盖之前的值；
- 命令行传入的 `.conf` 会覆盖这个默认路径。
- **必须在 `flag::parse` 前调用**；

### set_program_version

- 设置程序版本号；
- **必须在 `flag::parse` 前调用**，若在 `flag::parse` 后调用，`./xx -version` 无法显示版本信息。

### hide / unhide

- `hide` 隐藏 flag，使其不出现在帮助信息和 `-mkconf` 生成的配置文件中；
- `unhide` 与 `hide` 相反；
- **必须在 `flag::parse` 前调用**。

### set_value

- 按名字设置 flag 的值，`value` 是字符串形式，内部按 flag 类型解析；
- 出错时返回 `false`，并用 `co::println` 打印错误信息，不会退出程序；
- **通常在 `flag::parse` 前调用**，用于修改 flag 默认值，这样命令行与配置文件中传入的值依旧可以覆盖它；
- 若在 `flag::parse` 后调用，命令行、配置文件中的设置将失去作用，一般不建议这样做。

### run_after_parse

- 注册的回调在 `flag::parse` 解析完命令行参数与配置文件后执行；
- 允许注册多个回调，按注册顺序执行；
- coost 内部用它启动协程调度线程、日志线程。
- **必须在 `flag::parse` 前调用**。

### run_before_parse

- 注册的回调在 `flag::parse` 解析参数前执行；
- 允许注册多个回调，按注册顺序执行；
- coost 内部用它来 unhide 组件中的 flag，例如包含 `co/rpc.h` 后，RPC 组件会在回调中调用 `flag::unhide("rpc_max_msg_size")` 等。
- **必须在 `flag::parse` 前调用**。


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
