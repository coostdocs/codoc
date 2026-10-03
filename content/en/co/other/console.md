---
weight: 1
title: "Console Output"
---

## Header

```cpp
#include "co/print.h"
```

The API is in the `co` namespace.

## API

```cpp
template<typename ...X>
co::xx::stream& co::print(X&& ...x);

template<typename ...X>
void co::println(X&& ...x);
```

- Outputs to `stderr`; both are **thread-safe** and accept any number of arguments.
- Arguments can be any type supported by `co::string::operator<<`;
- `co::print` returns `xx::stream&`, allowing chained calls to `.flush()`.
- `co::println` automatically adds a newline.
- `co::print` and `co::println` use different buffers.

{{< hint warning >}}
`co::print` flushes the buffer when the data reaches a certain amount or when the user manually calls `.flush()`; `co::println` flushes the buffer immediately.
{{< /hint >}}

## Colors

`print` and `println` support the following colors (foreground only):

```cpp
co::color::deflt                # terminal default color
co::color::red
co::color::green
co::color::yellow
co::color::blue
co::color::magenta              # purple
co::color::cyan                 # cyan
co::color::bold                 # bold
co::color::bright_red
co::color::bright_green
co::color::bright_yellow
co::color::bright_blue
co::color::bright_magenta
co::color::bright_cyan
```

{{< hint warning >}}
If `stderr` is redirected to a file, color output is automatically disabled.
{{< /hint >}}

`co::color` has two usages:

- With content: writes the color sequence + content + restores the default color:

```cpp
co::color c;
co::println(c.red("red")); // c.red is equivalent to co::color::red
```

- As a stream manipulator: the color is passed in as a function pointer, only the color sequence is written, and `c.deflt` must be passed manually to restore the default color:

```cpp
co::color c;
co::println("hello ", c.red, "coost ", 23, c.deflt);
```

## Example

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

## Notes on Mixing print and println

`co::print` and `co::println` use **independent thread-local buffers**. When the two are mixed, output to the terminal is not necessarily in code order, for example:

```cpp
co::print("a");
co::println("b");
```

`println` flushes the buffer immediately, so `"b"` may be output before `"a"`.
