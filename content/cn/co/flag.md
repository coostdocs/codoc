---
weight: 5
title: "命令行参数与配置文件解析"
---

include: [co/flag.h](https://github.com/idealvin/co/blob/master/include/co/flag.h).


## 基本概念

`co/flag` 是一个类似 [google gflags](https://github.com/gflags/gflags) 的命令行参数及配置文件解析库，其原理很简单，代码中定义全局变量，然后在程序启动时解析命令行参数或配置文件，修改这些全局变量的值。



### flag 变量

co/flag 中的宏定义的全局变量，称为 **flag 变量**。如下面的代码定义了一个 flag 变量，变量名是 `FLG_x`。

```cpp
DEF_bool(x, false, "xxx"); // bool FLG_x = false;
```

co/flag 支持 7 种类型的 flag 变量：

```cpp
bool, int32, int64, uint32, uint64, double, string
```

每个 flag 变量都有一个默认值，用户可以通过命令行参数或配置文件修改 flag 变量的值。如前面定义的 `FLG_x`，在命令行中可以用 `-x=true`，在配置文件中可以用 `x = true`，设置一个新的值。



### command line flag

命令行参数中，以 `-x=y` 的形式出现，其中 `x` 被称为一个 **command line flag**(以下简称为 flag)。命令行中的 flag `x` 对应代码中的全局变量 `FLG_x`，命令行中的 `-x=y` 就相当于将 `FLG_x` 的值设置为 `y`。为了方便，下面可能将 flag 与 flag 变量统一称为 flag。

co/flag 为了简便易用，设计得非常灵活：

- `-x=y` 可以省略前面的 `-`，简写为 `x=y`.
- `-x=y` 也可以写成 `-x y`.
- `x=y` 前面可以添加任意数量的 `-`.
- bool 类型的 flag，`-b=true` 可以简写为 `-b`.

- 示例

```bash
# b, i, s 都是 flag, xx 不是 flag
./exe -b -i=32 -s=hello xx
```




## API

### flag::init

```cpp
co::vector<fastring> init(int argc, const char** argv);
co::vector<fastring> init(int argc, char** argv);
void init(const fastring& path);
```

- 前两个 init 函数，解析命令行参数及配置文件，并更新 flag 变量的值。**此函数一般需要在 main 函数开头调用一次**。大致流程如下：
  - 对命令行参数进行预处理，此过程中可能会更新 `FLG_config` 的值。
  - 如果 `FLG_config` 非空，解析由它指定的配置文件，更新 flag 变量的值。
  - 解析其他命令行参数，更新 flag 变量的值。
  - 若 `FLG_mkconf` 为 true，则生成配置文件，并退出程序。
  - 若 `FLG_daemon` 为 true，则将程序放入后台运行 (仅适用于 linux 平台)。
  - 遇到任何错误时，输出错误信息，并立即退出程序。
  - 若未发生任何错误，返回 non-flag 列表。如执行 `./exe x y` 时，此函数将返回 `["x", "y"]`。

- 最后一个 init 函数，解析配置文件，并更新 flag 变量的值。参数 `path` 是配置文件的路径。遇到错误时，会输出错误信息，并退出程序。


- 示例

```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);
}
```



### flag::set_value

```cpp
fastring set_value(const fastring& name, const fastring& value)
```

- v3.0 新增，设置 flag 变量的值，`name` 是 flag 名。
- 此函数非线程安全，一般需要在 main 函数开头调用。


- 示例

```cpp
DEF_bool(b, false, "");
DEF_int32(i, 0, "");
DEF_string(s, "", "");
flag::set_value("b", "true"); // FLG_b -> true
flag::set_value("i", "23");   // FLG_i -> 23
flag::set_value("s", "xx");   // FLG_s -> "xx"
```



### flag::alias

```cpp
bool alias(const char* name, const char* new_name);
```

- v3.0 新增，给 flag 取别名，在**命令行参数或配置文件中**可以用别名取代原名。
- 此函数非线程安全，需要在 `flag::init()` 之前调用。


- 示例

```cpp
DEF_bool(all, false, "");

int main(int argc, char** argv) {
    flag::alias("all", "a");
    flag::init(argc, argv);
}
```




## 代码中使用 flag 变量


### 定义 flag 变量

```cpp
DEF_bool(name, value, help, ...)
DEF_int32(name, value, help, ...)
DEF_int64(name, value, help, ...)
DEF_uint32(name, value, help, ...)
DEF_uint64(name, value, help, ...)
DEF_double(name, value, help, ...)
DEF_string(name, value, help, ...)
```

- 上面的 7 个宏，分别用于定义 7 种不同类型的 flag 变量。
- 参数 `name` 是 flag 名，对应的全局变量名是 `FLG_name`，参数 `value` 是默认值，参数 `help` 是注释信息。
- flag 变量是全局变量，一般不要在头文件中定义。
- flag 变量的名字是唯一的，不能定义两个名字相同的 flag 变量。
- flag 变量一般在命名空间之外定义，否则可能无法使用 FLG_name 访问 flag 变量。


- 示例

```cpp
DEF_bool(b, false, "comments"); // bool FLG_b = false;
DEF_int32(i32, 32, "comments"); // int32 FLG_i32 = 32;
DEF_int64(i64, 64, "comments"); // int64 FLG_i64 = 64;
DEF_uint32(u32, 0, "comments"); // uint32 FLG_u32 = 0;
DEF_uint64(u64, 0, "comments"); // uint64 FLG_u64 = 0;
DEF_double(d, 2.0, "comments"); // double FLG_d = 2.0;
DEF_string(s, "x", "comments"); // fastring FLG_s = "x";
```



### flag 添加别名

- v3.0 新增，定义 flag 变量时，可以为 flag 添加任意数量的别名。
- 在命令行或配置文件中，可以用别名取代原名。


- 示例

```cpp
DEF_bool(debug, false, "");         // no alias
DEF_bool(debug, false, "", d);      // d is an alias of debug
DEF_bool(debug, false, "", d, dbg); // 2 aliases
```



### 声明 flag 变量

```cpp
DEC_bool(name)
DEC_int32(name)
DEC_int64(name)
DEC_uint32(name)
DEC_uint64(name)
DEC_double(name)
DEC_string(name)
```

- 上面的 7 个宏，分别用于声明 7 种不同类型的 flag 变量。
- 参数 `name` 是 flag 名，对应的全局变量名是 `FLG_name`。
- 一个 flag 变量只能定义一次，但可以声明多次，可以在任何需要的地方声明它们。
- flag 变量一般在命名空间之外声明，否则可能无法使用 FLG_name 访问 flag 变量。


- 示例

```cpp
DEC_bool(b);     // extern bool FLG_b;
DEC_int32(i32);  // extern int32 FLG_i32;
DEC_int64(i64);  // extern int64 FLG_i64;
DEC_uint32(u32); // extern uint32 FLG_u32;
DEC_uint64(u64); // extern uint64 FLG_u64;
DEC_double(d);   // extern double FLG_d;
DEC_string(s);   // extern fastring FLG_s;
```



### 使用 flag 变量

定义或声明 flag 变量后，就可以像普通变量一样使用它们：

```cpp
#include "co/flag.h"

DEC_bool(b);
DEF_string(s, "hello", "xxx");

int main(int argc, char** argv) {
    flag::init(argc, argv);
    
    if (!FLG_b) std::cout << "b is false" << std::endl;
    FLG_s += " world";
    std::cout << FLG_s << std::endl;
    
    return 0;
}
```




## 命令行中使用 flag


### 修改 flag 变量的值

假设程序中定义了如下的 flag：

```cpp
DEF_bool(x, false, "bool x");
DEF_bool(y, false, "bool y");
DEF_int32(i, -32, "int32");
DEF_uint64(u, 64, "uint64");
DEF_string(s, "nice", "string");
```

程序启动时，可以通过命令行参数修改 flag 变量的值：

```bash
# -x=y, x=y, -x y, 三者是等价的
./xx -i 8 -u 88 -s "hello world"
./xx -i=8 u=88 -s=xxx
./xx -i8       # 仅适用于单字母命名的整数类型 flag

# bool 类型设置为 true 时, 可以略去值
./xx -x        # -x=true

# 多个单字母命名的 bool flag, 可以合并设置为 true
./xx -xy       # -x=true -y=true

# 整数类型的 flag 可以带单位 k, m, g, t, p, 不区分大小写
./xx -i -4k    # i=-4096

# 整数类型的 flag 可以传 8 进制 或 16 进制数
./xx i=032     # i=26     8 进制
./xx u=0xff    # u=255   16 进制
```



### 查看帮助信息(--help)

co/flag 支持用 `--help` 命令查看程序的帮助信息，该命令会显示 usage 信息及用户定义的 flag 列表。

```bash
$ ./xx --help
usage:  $exe [-flag] [value]
        $exe -x -i 8k -s ok        # x=true, i=8192, s="ok"
        $exe --                    # print all flags
        $exe -mkconf               # generate config file
        $exe -conf xx.conf         # run with config file

flags:
    -n  int32
        type: int32       default: 0
        from: test/flag.cc
    -s  string
        type: string      default: "hello world"
        from: test/flag.cc
```



### 查看 flag 列表(--)

co/flag 可以用 `--` 命令查看程序中定义的全部 flag 列表(包括co内部定义的flags)：

```bash
$ ./xx --
flags:
    -boo  bool flag
        type: bool        default: false
        from: test/flag.cc
    -co_debug_log  enable debug log for coroutine library
        type: bool        default: false
        from: src/co/scheduler.cc
    -co_sched_num  number of coroutine schedulers, default: os::cpunum()
        type: uint32      default: os::cpunum()
        from: src/co/scheduler.cc
```



### 查看程序版本信息

- `version` 是 co/flag 内部定义的 flag，命令行中可以使用 `--version` 命令查看版本信息。
- `version` 默认值为空，用户需要在调用 `flag::init()` 前，修改其值。


- 示例

```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
    FLG_version = "v3.0.0";
    flag::init(argc, argv);
    return 0;
}
```

```bash
$ ./xx --version
v3.0.0
```




## 配置文件


### 配置文件格式

co/flag 的配置文件格式比较灵活：

- 一行一个配置项，每个配置项对应一个 flag，形式统一为 `x = y`，看起来一目了然。
- `#` 或 `//` 表示注释，支持行尾注释。
- 引号中的 `#` 或 `//`  不是注释。
- 忽略行前、行尾的空白字符，书写更自由，不容易出错。
- `=` 号前后可以任意添加空白字符，书写更自由。
- 可以用 `\` 续行，以免一行太长，影响美观。
- 字符串不支持转义，以免产生歧义。
- 字符串可以用双引号、单引号或 3个反引号括起来。


- 配置文件示例

```yaml
   # config file: xx.conf
     boo = true                # bool 类型

     s =                       # 空字符串
     s = hello \
         world                 # s = "helloworld"
     s = "http://github.com"   # 引号中的 # 与 // 不是注释
     s = "I'm ok"              # 字符串中含有单引号，两端可以用双引号括起来
     s = 'how are "U"'         # 字符串中含有双引号，两端可以用单引号括起来
     s = ```I'm "ok"```        # 字符串两端也可以用 3 个反引号括起来

     i32 = 4k                  # 4096, 整型可以带单位 k,m,g,t,p, 不区分大小写
     i32 = 032                 #  8 进制, i32 = 26
     i32 = 0xff                # 16 进制, i32 = 255
     pi = 3.14159              # double 类型
```



### 自动生成配置文件

- `mkconf` 是 co/flag 内部定义的 flag，它是自动生成配置文件的开关。
- 命令行中可以用 `-mkconf` 自动生成配置文件。

```bash
./xx -mkconf            # 在 xx 所在目录生成 xx.conf
./xx -mkconf -x u=88    # 自定义配置项的值
```



### 调整配置项的顺序

自动生成的配置文件中，配置项按 flag 级别、所在文件名、所在代码行数进行排序。如果用户想让某些配置项的排序靠前些，可以将 flag 的级别设成较小的值，反之可以将 flag 级别设成较大的值。

定义 flag 时可以在注释开头用 `#n` 指定级别，**n 必须是 0 到 9 之间的整数**，若注释非空，n 后面必须有一个空格。不指定时，默认 flag 级别为 5。

```cpp
DEF_bool(x, false, "comments");    // 默认级别为 5
DEF_bool(y, false, "#3");          // 级别为 3, 注释为空
DEF_bool(z, false, "#3 comments"); // 级别为 3, 注释非空, 3 后面必须有一个空格
```



### 禁止配置项生成到配置文件

注释以 `.` 开头的 flag，带有**隐藏**属性，不会生成到配置文件中，但用 `--` 命令可以查看。注释为空的 flag，则是完全不可见的，既不会生成到配置文件中，也不能用 `--` 命令查看。

```cpp
DEF_bool(x, false, ".say something here");
DEF_string(s, "good", "");
```



### 程序启动时指定配置文件

```cpp
DEF_string(config, "", ".path of config file", conf);
```

- `config` 是 co/flag 内部定义的 flag，表示配置文件的路径，它有一个别名 `conf`。
- 命令行中可以用 `-config` 或 `-conf` 指定配置文件。
- 代码中可以在调用 `flag::init()` 之前，修改 `FLG_config` 的值，以指定配置文件。

```bash
./xx -config xx.conf
./xx -conf xx.conf

# 若配置文件名以 .conf 或 config 结尾, 且是命令行的
# 第一个 non-flag 参数, 则可省略 -config
./xx xx.conf
./xx xx.conf -x
```




## 自定义帮助信息

- `help` 是 co/flag 内部定义的 flag，命令行中可以使用 `--help` 命令查看帮助信息。
- `FLG_help` 默认为空，使用 co/flag 内部提供的默认帮助信息。
- 用户想自定义帮助信息时，可以在调用 `flag::init()` 前，修改 `FLG_help` 的值。


- 示例

```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
    FLG_help << "usage:\n"
             << "\t./xx -ip 127.0.0.1 -port 7777\n";
    flag::init(argc, argv);
    return 0;
}
```




## 让程序在后台运行

- `daemon` 是 co/flag 内部定义的 flag，若为 true，程序将在后台运行，仅支持 linux 平台。

- 命令行中可以用 `-daemon` 指定程序以 daemon 形式在后台运行。


- 示例

```bash
./xx -daemon
```

