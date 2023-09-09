---
weight: 4
title: "flag"
---

include: [co/flag.h](https://github.com/idealvin/coost/blob/master/include/co/flag.h).


## Basic concepts

**co.flag** is a command line and config file parsing library. Its principle is very simple, define global variables in code, then parse the command line parameters and/or config file when the program starts, and update the value of these global variables. 



### flag variable

The global variable defined by macros in `co.flag` are called **flag variable**. For example, the following code defines a flag variable, the variable name is `FLG_x`.

```cpp
DEF_int32(x, 0, "xxx"); // int32 FLG_x = 0;
```

`co.flag` supports 7 types of flag variable:

```cpp
bool, int32, int64, uint32, uint64, double, string
```

Every flag variable has a default value, and a new value can be passed to it from command-line or config file. Take the previously `FLG_x` as an example, we can use `-x=23` in command line, or `x = 23` in the config file, to set a new value for it.



### command line flag

Command line parameters appear in the form of `-x=y`, where `x` is called a **command line flag** (hereinafter referred to as flag). The flag `x` in command line corresponds to the global variable `FLG_x` in the code, and `-x=y` in command line is equivalent to setting the value of `FLG_x` to `y`. 

`co.flag` is designed to be very flexible:

- `-x=y` can omit the preceding `-`, abbreviated as `x=y`.
- `-x=y` can also be written as `-x y`.
- `x=y` can be preceded by any number of `-`.
- For bool type flags, `-b=true` can be abbreviated as `-b`.

- Example

```bash
# b, i, s are all flags, xx is not a flag
./exe -b -i=32 -s=hello xx
```




## APIs

### flag::parse

```cpp
co::vector<fastring> parse(int argc, char** argv);
void parse(const fastring& path);
```

- Added in v3.0.1.
- The first parse function, parse the command line parameters and config file, and update value of the flag variables. It usually needs to be called once at the beginning of the main function. Generally speaking, it does the following steps:
  - Preprocess the command line parameters, the value of `FLG_config` may be updated then.
  - If `FLG_config` is not empty, parse the config file specified by it, and update value of the flag variables.
  - Parse other command line parameters and update value of the flag variables.
  - If `FLG_mkconf` is true, generate a config file and terminate the program.
  - If `FLG_daemon` is true, run the program as a daemon (for Linux only).
  - When any error occurs, print the error message and terminate the program immediately.
  - If no error occurs, return the non-flag list. For example, when executing `./exe x y`, this function will return `["x", "y"]`.

- The second parse function, parses the config file and updates value of the flag variables. The parameter `path` is the path of the config file. When any error occurs, print the error message and terminate the program.


{{< hint warning >}}
**flag::init**  
Since v3.0.1，`flag::init()` has been marked as deprecated, please use `flag::parse()` instead.
{{< /hint >}}


- Example

```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);
}
```



### flag::set_value

```cpp
fastring set_value(const fastring& name, const fastring& value)
```

- Added in v3.0. Set value of a flag variable, `name` is the flag name.
- This function is not thread-safe and usually needs to be called at the beginning of the main function.


- Example

```cpp
DEF_bool(b, false, "");
DEF_int32(i, 0, "");
DEF_string(s, "", "");

int main(int argc, char** argv) {
    flag::set_value("b", "true"); // FLG_b -> true
    flag::set_value("i", "23");   // FLG_i -> 23
    flag::set_value("s", "xx");   // FLG_s -> "xx"
    flag::parse(argc, argv);
}
```



### flag::alias

```cpp
bool alias(const char* name, const char* new_name);
```

- Added in v3.0. Add an alias to a flag, in **command line or config file** you can replace the original name with the alias.

- This function is not thread safe and needs to be called before `flag::parse()`.


- Example

```cpp
DEF_bool(all, false, "");

int main(int argc, char** argv) {
     flag::alias("all", "a");
     flag::parse(argc, argv);
}
```



## Use flag variable in the code


### Define a flag variable

```cpp
DEF_bool(name, value, help, ...)
DEF_int32(name, value, help, ...)
DEF_int64(name, value, help, ...)
DEF_uint32(name, value, help, ...)
DEF_uint64(name, value, help, ...)
DEF_double(name, value, help, ...)
DEF_string(name, value, help, ...)
```

- The above 7 macros are used to define 7 different types of flag variables.
- The parameter `name` is the flag name, the corresponding global variable name is `FLG_name`, the parameter `value` is the default value, and the parameter `help` is comment for the flag.
- A flag variable is a global variable and generally should not be defined in the header file.
- The name of the flag variable is unique, and we cannot define two flag variables with the same name.
- The flag variable is generally defined outside the namespace, otherwise it may be not possible to use FLG_name to access the flag variable.


- Example

```cpp
DEF_bool(b, false, "comments"); // bool FLG_b = false;
DEF_int32(i32, 32, "comments"); // int32 FLG_i32 = 32;
DEF_int64(i64, 64, "comments"); // int64 FLG_i64 = 64;
DEF_uint32(u32, 0, "comments"); // uint32 FLG_u32 = 0;
DEF_uint64(u64, 0, "comments"); // uint64 FLG_u64 = 0;
DEF_double(d, 2.0, "comments"); // double FLG_d = 2.0;
DEF_string(s, "x", "comments"); // fastring& FLG_s = ...;
```


{{< hint info >}}
Since v3.0.1, `DEF_string` actually defines a reference to `fastring`. 
{{< /hint >}}



### Add alias for a flag

- Added in v3.0, when defining a flag variable, you can add any number of aliases to the flag.
- In command line or config file, alias can be used instead of the original name.


- Example

```cpp
DEF_bool(debug, false, "");         // no alias
DEF_bool(debug, false, "", d);      // d is an alias of debug
DEF_bool(debug, false, "", d, dbg); // 2 aliases
```



### Declare the flag variable

```cpp
DEC_bool(name)
DEC_int32(name)
DEC_int64(name)
DEC_uint32(name)
DEC_uint64(name)
DEC_double(name)
DEC_string(name)
```

- The 7 macros above are used to declare 7 different types of flag variables.
- The parameter `name` is the flag name, and the corresponding global variable name is `FLG_name`.
- A flag variable can be defined only once, but it can be declared multiple times, which can be declared wherever needed.
- The flag variable is generally declared outside the namespace, otherwise it may be not possible to use FLG_name to access the flag variable.


- Example

```cpp
DEC_bool(b);     // extern bool FLG_b;
DEC_int32(i32);  // extern int32 FLG_i32;
DEC_int64(i64);  // extern int64 FLG_i64;
DEC_uint32(u32); // extern uint32 FLG_u32;
DEC_uint64(u64); // extern uint64 FLG_u64;
DEC_double(d);   // extern double FLG_d;
DEC_string(s);   // extern fastring& FLG_s;
```



### Use the flag variable

Once a flag variable is defined or declared, we can use it the same as an ordinary variable.

```cpp
#include "co/flag.h"

DEC_bool(b);
DEF_string(s, "hello", "xxx");

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    
    if (!FLG_b) std::cout << "b is false" << std::endl;
    FLG_s += " world";
    std::cout << FLG_s << std::endl;
    
    return 0;
}
```




## Use flag in command line


### Set value of flags

Suppose the following flags are defined in the program:

```cpp
DEF_bool(x, false, "bool x");
DEF_bool(y, false, "bool y");
DEF_int32(i, -32, "int32");
DEF_uint64(u, 64, "uint64");
DEF_string(s, "nice", "string");
```

When the program starts, we can modify value of the flag variables through command line parameters:

```bash
# -x=y, x=y, -x y, the three are equivalent
./xx -i=8 u=88 -s=xxx
./xx -i 8 -u 88 -s "hello world"
./xx -i8       # -i=8, only for single-letter named integer flags

# When a bool type is set to true, the value can be omitted
./xx -x        # -x=true

# Multiple single-letter named bool flags can be combined and set to true
./xx -xy       # -x=true -y=true

# Integer type flags can have units k, m, g, t, p, not case sensitive
./xx -i -4k    # i=-4096

# Integer type flags can pass octal or hexadecimal numbers
./xx i=032     # i=26 octal
./xx u=0xff    # u=255 hexadecimal
```



### Show Help Information

`co.flag` supports using `--help` command to print the help information of the program:

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



### List all flags

`co.flag` provides `--` command to list all the flags defined in the program:

```bash
$ ./xx --
flags:
    -boo  bool flag
        type: bool        default: false
        from: test/flag.cc
    -co_sched_num  number of coroutine schedulers, default: os::cpunum()
        type: uint32      default: os::cpunum()
        from: src/co/sched.cc
```



### Show version of the program

- `version` is a flag defined inside coost. You can use the `-version` command to print version information.
- `version` is empty by default, its value should be set before calling `flag::parse()`.


- Example

```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
     FLG_version = "v3.0.0";
     flag::parse(argc, argv);
     return 0;
}
```

```bash
$ ./xx -version
v3.0.0
```




## config file


### config file format

The config file format of `co.flag` is flexible:

- One config item per line, each config item corresponds to a flag, and the form is unified as `x = y`, which looks clear at a glance.
- `#` or `//` are for comments.
- `#` or `//` in quotation marks are not comments.
- Ignore the blank characters at the beginning or end of the line.
- Blank characters can be added before or after the `=` sign.
- `\` can be used to continue a line to avoid too long a line.
- The string does not support escaping to avoid ambiguity.
- The string can be enclosed in double quotes, single quotes or 3 back quotes.


- Sample config file

```yaml
   # config file: xx.conf
     boo = true                 # bool type

     s =                        # empty string
     s = hello \
         world                  # s = "helloworld"
     s = "http://github.com"    # # or // in quotation marks are not comments
     s = "I'm ok"               # enclose the string in double quotes
     s ='how are "U"'           # enclose the string in single quotes
     s = ```I'm "ok"```         # enclose the string in 3 back quotes

     i32 = 4k                   # 4096, integers can have units k, m, g, t, p, not case sensitive
     i32 = 032                  # octal, i32 = 26
     i32 = 0xff                 # hexadecimal, i32 = 255
     pi = 3.14159               # double type
```



### Generate config file

- `mkconf` is a flag defined internally in coost, which is a switch for automatically generating config file.
- You can use `-mkconf` to generate a config file in command line.

```bash
./xx -mkconf            # Generate xx.conf
./xx -mkconf -x u=88    # Custom config item value
```



### Adjust the order of config items

In the automatically generated config file, the config items are sorted by flag level, file name, and code line number. If the user wants some config items to be ranked higher, the flag level can be set to a smaller value, otherwise, the flag level can be set to a larger value. 

When defining a flag, you can use `#n` at the beginning of the comment to specify the flag level, **n must be an integer between 0 and 9**. If the comment is not empty, there must be a space after n. When not specified, the default flag level is 5.

```cpp
DEF_bool(x, false, "comments");    // The default level is 5
DEF_bool(y, false, "#3");          // The level is 3, and the comment is empty
DEF_bool(z, false, "#3 comments"); // The level is 3
```



### Prohibit config items from being generated in the config file

Flags whose comments start with `.`, are **hidden flags**, which will not be present in the config file, but can be found with the `--` command in command line. A flag with an empty comment is completely invisible and will neither be generated in the config file nor be found with the `--` command.

```cpp
DEF_bool(x, false, ".say something here");
DEF_string(s, "good", "");
```



### Specify the config file when the program starts

- `config` is a flag defined internally in coost, which is the path of the config file. It has an alias `conf`.
- You can use `-config` to specify the config file in command line.
- Another way, you can modify the value of `FLG_config` to specify the config file, before calling `flag::parse()`.

```bash
./xx -config xx.conf
./xx -conf xx.conf

# If the config file name ends with .conf or config, 
# and it is the first non-flag parameter in command line, 
# -config can be omitted.
./xx xx.conf
./xx xx.conf -x
```




## Custom help information

- `help` is a flag defined in coost, which stores the help information of the program. This information can be seen with the command `--help` in command line.
- `FLG_help` is empty by default, and the default help information provided by coost is used.
- You can modify the value of `FLG_help` before calling `flag::parse()` to customize the help information.


- Example

```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
    FLG_help << "usage:\n"
             << "\t./xx -ip 127.0.0.1 -port 7777\n";
    flag::parse(argc, argv);
    return 0;
}
```




## Run program as a daemon

- `daemon` is a flag defined in coost. If it is true, the program will run as a daemon. It only works on Linux platform.
- You can use `-daemon` in command line to make the program run in the background as a daemon.

- Example

```bash
./xx -daemon
```
