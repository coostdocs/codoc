---
weight: 1
title: "终端输出"
---

## 头文件

```cpp
#include "co/print.h"
```

API 在 `co` 命名空间。


## API

```cpp
template<typename ...X>
co::xx::stream& co::print(X&& ...x);

template<typename ...X>
void co::println(X&& ...x);
```

- 输出到 `stderr`，均**线程安全**，接受任意数量参数。
- 参数可以是 `co::string::operator<<` 支持的任意类型；
- `co::print` 返回 `xx::stream&`，可链式调用 `.flush()`。
- `co::println` 自动添加换行。
- `co::print` 与 `co::println` 使用不同的缓冲区。

{{< hint warning >}}
`co::print` 在数据达到一定量或用户手动调用 `.flush()` 时刷新缓冲区，`co::println` 则是立即刷新缓冲区。
{{< /hint >}}


## 颜色

`print`、`println` 支持如下颜色(仅前景色):

```cpp
co::color::deflt                # 终端默认颜色
co::color::red
co::color::green
co::color::yellow
co::color::blue
co::color::magenta              # 紫
co::color::cyan                 # 青
co::color::bold                 # 粗体
co::color::bright_red
co::color::bright_green
co::color::bright_yellow
co::color::bright_blue
co::color::bright_magenta
co::color::bright_cyan
```

{{< hint warning >}}
若 `stderr` 重定向到文件，会自动关闭颜色输出。
{{< /hint >}}

`co::color` 有两种用法：

- 带内容：写入颜色序列 + 内容 + 恢复默认颜色:

```cpp
co::color c;
co::println(c.red("red")); // c.red 与 co::color::red 等价
```

- 作为流式控制符：颜色作为函数指针传入，只写颜色序列，需手动传 `c.deflt` 恢复默认颜色:

```cpp
co::color c;
co::println("hello ", c.red, "coost ", 23, c.deflt);
```


## 示例

```cpp
#include "co/print.h"

int main(int argc, char** argv) {
    co::color c;
    co::print(c.red("red\n"));
    co::print(c.green("green\n"));
    co::print(c.yellow("yellow\n"));
    co::print(c.blue("blue\n"));
    co::print(c.magenta("magenta\n"));
    co::print(c.cyan("cyan\n"));
    co::print(c.bold("bold\n"));
    co::print(c.bright_red("bright_red\n"));
    co::print(c.bright_green("bright_green\n"));
    co::print(c.bright_yellow("bright_yellow\n"));
    co::print(c.bright_blue("bright_blue\n"));
    co::print(c.bright_magenta("bright_magenta\n"));
    co::print(c.bright_cyan("bright_cyan\n"));

    co::print("hello ", c.red, "coost ", 23, '\n', c.deflt).flush();
    co::println("hello ", c.red, "coost ", c.green, 123, c.deflt);
    return 0;
}
```

## 混用 print 与 println 注意事项

`co::print` 和 `co::println` 使用**独立线程局部缓冲区**，二者混用时，不一定按代码顺序输出到终端，如：

```cpp
co::print("a");
co::println("b");
```

`println` 立即刷新缓冲区，`"b"` 可能在 `"a"` 之前输出。
