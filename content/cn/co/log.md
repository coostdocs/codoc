---
weight: 6
title: "日志"
---

include: [co/log.h](https://github.com/idealvin/co/blob/master/include/co/log.h).


## 简介

`co/log` 是一个类似 [google glog](https://github.com/google/glog) 的 C++ 流式日志库，它像下面这样打印日志：

```cpp
LOG << "hello world" << 23;     // level log
TLOG("topic") << "hello" << 23; // topic log
```

co/log 支持两种类型的日志，level log 与 topic log(TLOG)。



###  Level Log 

Level log 分为 debug, info, warning, error, fatal 5 个级别，并提供一系列的宏，用于打印不同级别的日志。

**打印 fatal 级别的日志会终止程序的运行**，co/log 还会在程序退出前打印函数调用栈信息，以方便追查程序崩溃的原因。

此类型的日志会写入同一个文件中，一般用于打印调试信息。



### Topic Log

Topic log(TLOG) 没有级别之分，而是按主题分类。

此类型的日志，按主题写入不同的文件，一般用于打印业务日志，按业务功能划分，便于日志管理。



### 性能

co/log 内部采用异步的实现方式，日志先写入缓存，达到一定量或超过一定时间后，由后台线程一次性写入文件，性能在不同平台比 glog 提升了 20~150 倍左右。下表是不同平台单线程连续打印 100 万条(每条 50 字节左右) info 日志的测试结果：

| platform | google glog | co/log |
| --- | --- | --- |
| win2012 HHD | 1.6MB/s | 180MB/s |
| win10 SSD | 3.7MB/s | 560MB/s |
| mac SSD | 17MB/s | 450MB/s |
| linux SSD | 54MB/s | 1023MB/s |




## API

### log::exit

```cpp
void exit();
```

- 将缓存中的日志写入文件，并退出后台写日志的线程。
- 程序正常退出时，co/log 会自动调用此函数。
- 多次调用此函数是安全的。
- co/log 内部捕获到 `SIGINT, SIGTERM, SIGQUIT` 等信号时，会在程序退出前调用此函数。



### log::set_write_cb

```cpp
void set_write_cb(
    const std::function<void(const void*, size_t)>& cb,
    int flags=0
);
void set_write_cb(
    const std::function<void(const char*, const void*, size_t)>& cb,
    int flags=0
);
```

- co/log 默认将日志写到本地文件，用户可以调用此 API 自定义写日志的 callback，将日志写到不同的目标。
- 参数 `cb` 是 callback。第 1 个版本用于 Level log，cb 带 2 个参数，表示日志 buffer 的地址及长度；第 2 个版本用于 Topic log(TLOG)，cb 带 3 个参数，第 1 个参数是 topic，后两个参数与第 1 个版本相同。buffer 中可能含有多条日志。
- 参数 `flags` 是 v3.0 新增，默认为 0，可以是下述选项的组合：
  - `log::log2local`，在本地也写一份日志。



### v3.0 删除的 API

- **log::init**，v3.0 移除，从 co 3.0 开始，一般只需要在 main 函数开头调用 `flag::init(argc, argv)`。

- **log::set_single_write_cb**，v3.0 移除。

- **log::close**，v3.0 移除，使用 `log::exit()` 取代之。




## Level Log


### 基本用法

```cpp
DLOG  LOG  WLOG  ELOG  FLOG
```

- 上面的 5 个宏分别用于打印 5 种不同级别的日志，它们是**线程安全**的。
- 这些宏实际上是 [fastream](../fastream/) 的引用，因此可以打印 `fastream::operator<<` 支持的任何类型。
- 这些宏会自动在每条日志末尾加上 '\n' 换行符，用户无需手动输入换行符。
- 前 4 种仅在 `FLG_min_log_level` 不大于当前日志级别时，才会打印日志，用户可以将 FLG_min_log_level 的值设置大一些，以屏蔽低级别的日志。
- 打印 fatal 级别的日志，表示程序出现了致命错误，co/log 会打印当前线程的函数调用栈信息，并终止程序的运行。


- 示例

```cpp
DLOG << "this is DEBUG log " << 23;
LOG << "this is INFO log " << 23;
WLOG << "this is WARNING log " << 23;
ELOG << "this is ERROR log " << 23;
FLOG << "this is FATAL log " << 23;
```



### 条件日志

```cpp
#define DLOG_IF(cond) if (cond) DLOG
#define  LOG_IF(cond) if (cond) LOG
#define WLOG_IF(cond) if (cond) WLOG
#define ELOG_IF(cond) if (cond) ELOG
#define FLOG_IF(cond) if (cond) FLOG
```

- 上面的 5 个宏，接受一个条件参数 cond，当 cond 为 true 时才打印日志。
- 参数 cond 可以是值为 bool 类型的任意表达式。
- 由于条件判断在最前面，即使相应级别的日志被屏蔽掉，这些宏也会保证 cond 表达式被执行。


- 示例

```cpp
int s = socket();
DLOG_IF(s != -1) << "create socket ok: " << s;
 LOG_IF(s != -1) << "create socket ok: " << s;
WLOG_IF(s == -1) << "create socket ko: " << s;
ELOG_IF(s == -1) << "create socket ko: " << s;
FLOG_IF(s == -1) << "create socket ko: " << s;
```



### 每 N 条打印一次日志

```cpp
#define DLOG_EVERY_N(n) _LOG_EVERY_N(n, DLOG)
#define  LOG_EVERY_N(n) _LOG_EVERY_N(n, LOG)
#define WLOG_EVERY_N(n) _LOG_EVERY_N(n, WLOG)
#define ELOG_EVERY_N(n) _LOG_EVERY_N(n, ELOG)
```

- 上面的宏每 n 条打印一次日志，内部用[原子操作](../atomic/)计数，是**线程安全**的。
- 参数 n 必须是大于 0 的整数，一般不要超过 int 类型的最大值。
- 第 1 条日志始终会被打印，之后每隔 n 条打印一次。
- fatal 日志一打印，程序就会终止运行，因此没有提供 FLOG_EVERY_N。


- 示例

```cpp
// 每 32 条打印一次 (1,33,65...)
DLOG_EVERY_N(32) << "this is DEBUG log " << 23;
LOG_EVERY_N(32) << "this is INFO log " << 23;
WLOG_EVERY_N(32) << "this is WARNING log " << 23;
ELOG_EVERY_N(32) << "this is ERROR log " << 23;
```



### 打印前 N 条日志

```cpp
#define DLOG_FIRST_N(n) _LOG_FIRST_N(n, DLOG)
#define  LOG_FIRST_N(n) _LOG_FIRST_N(n, LOG)
#define WLOG_FIRST_N(n) _LOG_FIRST_N(n, WLOG)
#define ELOG_FIRST_N(n) _LOG_FIRST_N(n, ELOG)
```

- 上面的宏打印前 n 条日志，内部用[原子操作](../atomic/)计数，是**线程安全**的。
- 参数 n 是不小于 0 的整数(等于 0 时不会打印日志)，一般不要超过 int 类型的最大值。
- 参数 n 一般不要用复杂的表达式，因为表达式 n 可能被执行两次。
- fatal 日志一打印，程序就会终止运行，因此没有提供 FLOG_FIRST_N。


- 示例

```cpp
// 打印前 10 条日志
DLOG_FIRST_N(10) << "this is DEBUG log " << 23;
LOG_FIRST_N(10) << "this is INFO log " << 23;
WLOG_FIRST_N(10) << "this is WARNING log " << 23;
ELOG_FIRST_N(10) << "this is ERROR log " << 23;
```




## TLOG

```cpp
#define TLOG(topic)
#define TLOG_IF(topic, cond) if (cond) TLOG(topic)
```

- TLOG 宏带有一个参数 `topic`，它是一个 C 风格的字符串，**需要具备静态生命周期**。
- TLOG_IF 宏仅当 cond 条件成立时，才打印日志。


- 示例

```cpp
TLOG("xx") << "hello " << 23;
TLOG_IF("xx", true) << "hello " << 23;
```




## CHECK 断言

```cpp
#define CHECK(cond) \
    if (!(cond)) _FLOG_STREAM << "check failed: " #cond "! "

#define CHECK_NOTNULL(p) \
    if ((p) == 0) _FLOG_STREAM << "check failed: " #p " mustn't be NULL! "

#define CHECK_EQ(a, b) _CHECK_OP(a, b, ==)
#define CHECK_NE(a, b) _CHECK_OP(a, b, !=)
#define CHECK_GE(a, b) _CHECK_OP(a, b, >=)
#define CHECK_LE(a, b) _CHECK_OP(a, b, <=)
#define CHECK_GT(a, b) _CHECK_OP(a, b, >)
#define CHECK_LT(a, b) _CHECK_OP(a, b, <)
```

- 上面的宏可视为加强版的 assert，它们在 DEBUG 模式下也不会被清除。
- 这些宏与 `FLOG` 类似，可以打印 fatal 级别的日志。
- `CHECK` 断言条件 cond 为真，cond 可以是值为 bool 类型的任意表达式。
- `CHECK_NOTNULL` 断言指针不是 NULL，参数 p 是任意类型的指针。
- `CHECK_EQ` 断言 `a == b`，参数 a 与 b 确保只计算一次。
- `CHECK_NE` 断言 `a != b`，参数 a 与 b 确保只计算一次。
- `CHECK_GE` 断言 `a >= b`，参数 a 与 b 确保只计算一次。
- `CHECK_LE` 断言 `a <= b`，参数 a 与 b 确保只计算一次。
- `CHECK_GT` 断言 `a > b`，参数 a 与 b 确保只计算一次。
- `CHECK_LT` 断言 `a < b`，参数 a 与 b 确保只计算一次。
- 一般建议优先使用 `CHECK_XX(a, b)` 这些宏，它们提供比 `CHECK(cond)` 更多的信息，会打印出参数 a 与 b 的值。
- `fastream::operator<<` 不支持的参数类型，如 STL 容器的 iterator 类型，不能用 `CHECK_XX(a, b)` 这些宏。
- 断言失败时，co/log 先调用 `log::exit()`，再打印当前线程的函数调用栈信息，然后退出程序。


- 示例

```cpp
int s = socket();
CHECK(s != -1);
CHECK(s != -1) << "create socket failed";
CHECK_NE(s, -1) << "create socket failed";  // s != -1
CHECK_GE(s, 0) << "create socket failed";   // s >= 0
CHECK_GT(s, -1) << "create socket failed";  // s > -1

std::map<int, int> m;
auto it = m.find(3);
CHECK(it != m.end()); // 不能使用 CHECK_NE(it, m.end()), 编译器会报错
```




## 打印堆栈信息

`co/log` 在 `CHECK` 断言失败或捕获到 `SIGSEGV` 等异常信号时，会打印函数调用栈，以方便定位问题，效果见下图：

![stack](https://coostdocs.github.io/images/stack.png)(https://asciinema.org/a/435894)

此功能需要在编译时，加入调试信息，如 gcc 需要开启 `-g` 选项。在 linux 与 macosx 平台，还需要安装 [libbacktrace](https://github.com/ianlancetaylor/libbacktrace)，才能打印堆栈信息。在 linux 上，`libbacktrace` 可能已经集成到 gcc 里了，您也许可以在类似 `/usr/lib/gcc/x86_64-linux-gnu/9` 的目录中找到它。否则，您可以按下面的方式手动安装它：

```sh
git clone https://github.com/ianlancetaylor/libbacktrace.git
cd libbacktrace-master
./configure
make -j8
sudo make install
```




## 配置项

co/log 使用 [co/flag](../flag/) 定义配置项，下面列出的是 co/log 内部定义的 flag，这些配置项如无特别说明，对 level log 与 TLOG 均有效。


### log_dir

```cpp
DEF_string(log_dir, "logs", "#0 log dir, will be created if not exists");
```

- 指定日志目录，默认为当前目录下的 `logs` 目录，不存在时将会自动创建。
- log_dir 可以是绝对路径或相对路径，路径分隔符可以是 '/' 或 '\\'，一般建议使用 '/'。
- 程序启动时，确保当前用户有足够的权限，否则创建日志目录可能失败。



### log_file_name

```cpp
DEF_string(log_file_name, "", "#0 name of log file, using exename if empty");
```

- 指定日志文件名(不含路径)，默认为空，使用程序名作为日志文件名(程序名末尾的 `.exe` 会被去掉)，如程序 xx 或 xx.exe 对应的日志文件名是 xx.log。
- 如果日志文件名不是以 `.log` 结尾，co/log 自动在文件名末尾加上 `.log`。



### min_log_level

```cpp
DEF_int32(min_log_level, 0, "#0 write logs at or above this level, 0-4 (debug|info|warning|error|fatal)");
```

- 仅适用于 level log，指定打印日志的最小级别，用于屏蔽低级别的日志，默认为 0，打印所有级别的日志。



### max_log_size

```cpp
DEF_int32(max_log_size, 4096, "#0 max size of a single log");
```

- 单条日志的最大长度，超过这个值，日志会被截断。



### max_log_file_size

```cpp
DEF_int64(max_log_file_size, 256 << 20, "#0 max size of log file, default: 256MB");
```

- 指定日志文件的最大大小，默认 256M，超过此大小，生成新的日志文件，旧的日志文件会被重命名。



### max_log_file_num

```cpp
DEF_uint32(max_log_file_num, 8, "#0 max number of log files");
```

- 指定日志文件的最大数量，默认是 8，超过此值，删除旧的日志文件。



### max_log_buffer_size

```cpp
DEF_uint32(max_log_buffer_size, 32 << 20, "#0 max size of log buffer, default: 32MB");
```

- 指定日志缓存的最大大小，默认 32M，超过此值，丢掉一半的日志。



### log_flush_ms

```cpp
DEF_uint32(log_flush_ms, 128, "#0 flush the log buffer every n ms");
```

- 后台线程将日志缓存刷入文件的时间间隔，单位为毫秒。



### log_daily

```cpp
DEF_bool(log_daily, false, ">>#0 if true, enable daily log rotation");
```

- 按天生成日志文件，默认为 false。



### cout

```cpp
DEF_bool(cout, false, "#0 also logging to terminal");
```

- 终端日志开关，默认为 false。若为 true，将日志也打印到终端。




## 日志文件


### 日志组织方式

co/log 将所有级别的日志记录到同一个文件中，默认使用程序名作为日志文件名，如进程 xx 的日志文件是 xx.log。当日志文件达到最大大小 (FLG_max_log_file_size) 时，co/log 会重命名日志文件，并生成新文件。日志目录下面可能包含如下的文件：

```bash
xx.log
xx_0523_16_12_54.970.log
xx_0523_16_13_12.921.log
xx_0523_16_15_05.264.log
```

`xx.log` 始终是当前最新的日志文件。当文件数量超过 `FLG_max_log_file_num` 时，co/log 就会删除最旧的日志文件。

`fatal` 级别的日志，还会额外记录到 `xx.fatal` 文件中，**co/log 不会重命名 fatal 日志文件，也不会删除它**。



### 日志格式

```bash
I0514 11:15:30.123 1045 test/xx.cc:11] hello world
D0514 11:15:30.123 1045 test/xx.cc:12] hello world
W0514 11:15:30.123 1045 test/xx.cc:13] hello world
E0514 11:15:30.123 1045 test/xx.cc:14] hello world
F0514 11:15:30.123 1045 test/xx.cc:15] hello world
```

- 上面的例子中，每行对应一条日志。
- 每条日志的第一个字母是日志级别，`I` 表示 info，`D` 表示 debug，`W` 表示 warning，`E` 表示 error，`F` 表示 fatal。
- 级别后面是时间，从月份开始到毫秒。年份意义不大，只会多占几个字节，因此没有打印出来。co/log 日志的时间，并不是生成时的时间，而是写入缓存时的时间，这是为了保证日志文件中的日志严格按时间排序。
- 时间后面是线程 id，上面的 1045 即是线程 id。
- 线程 id 后面是日志代码所在文件及行数。
- 行数后面是 `] ` ，即 `]` 后面加一个空格。
- `] ` 后面就是用户输入的日志内容。



### 查看日志

Linux 等系统，可以用 `grep`，`tail` 等命令查看日志。

```bash
grep ^E xx.log
tail -F xx.log
tail -F xx.log | grep ^E
```

- 第 1 行用 `grep` 过滤出文件中的错误日志，`^E` 表示以字母 `E` 开头。
- 第 2 行用 `tail -F` 命令动态追踪日志文件，这里需要用大写的 `F`，因为 xx.log 可能被重命名，然后生成新的 xx.log 文件，`-F` 确保按文件名追踪到最新的日志文件。
- 第 3 行用 `tail -F` 配合 `grep` 动态追踪日志文件中的错误日志。




## 构建及运行 co/log 测试程序

```bash
xmake -b log                   # build log or log.exe
xmake r log                    # run log or log.exe
xmake r log -cout              # also log to terminal
xmake r log -min_log_level=1   # 0-4: debug,info,warning,error,fatal 
xmake r log -perf              # performance test
```

- 在 co 根目录执行 `xmake -b log` 即可编译 [test/log.cc](https://github.com/idealvin/co/blob/master/test/log.cc) 测试代码，并生成 `log` 二进制文件。

