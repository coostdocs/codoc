---
weight: 5
title: "co/flag"
---

include: [co/flag.h](https://github.com/idealvin/co/blob/master/include/co/flag.h).


## Basic concepts


`co/flag` is a command line parameter and configuration file parsing library similar to [google gflags](https://github.com/gflags/gflags). Its principle is very simple, define global variables in code, then parse the command line parameters or configuration file when the program starts, and update the value of these global variables. 




### flag variable


The global variables defined by macros in co/flag are called **flag variables**. For example, the following code defines a flag variable, the variable name is `FLG_x`.


```cpp
DEF_bool(x, false, "xxx"); // bool FLG_x = false;
```


co/flag supports 7 types of flag variables:


```cpp
bool, int32, int64, uint32, uint64, double, string
```


Users can modify the value of the flag variable through command-line parameters or configuration files. Take the previously FLG_x as an example, in command line, we can use `-x=true` to modify its value, and in the configuration file, we can use `x = true` to set its value. 




### command line flag


Command line parameters appear in the form of `-x=y`, where `x` is called a **command line flag ** (hereinafter referred to as flag). The flag `x` in the command line corresponds to the global variable `FLG_x` in the code, and the `-x=y` in the command line is equivalent to setting the value of `FLG_x` to `y`. 


The flag library is designed to be very flexible:


- -x=y can omit the preceding -, abbreviated as x=y.
- -x=y can also be written as -x y.
- x=y can be preceded by any number of -.
- For bool type flags, -b=true can be abbreviated as -b.
- Example



```bash
# b, i, s are all flags, xx are not flags
./exe -b -i=32 -s=hello xx
```






## Initialization (flag::init)


```cpp
std::vector<fastring> init(int argc, char** argv);
```


- This function parses command line parameters and configuration file, and updates the value of flag variables. It needs to be called once at the beginning of the main function.
- argc, argv are the incoming parameters of the main() function.
- This function first parses the configuration file specified by `FLG_config`, if it is not empty, and then parses other command line parameters.
- If `FLG_mkconf` is true, generate the configuration file and terminate the program.
- If `FLG_daemon` is true, put the program to run in background (only for linux).
- When this function encounters an error, it prints the error message and exits the program immediately.
- If no error occurs, this function returns a non-flag list. For example, when executing `./exe x y`, flag::init() will return `["x", "y"]`.



- Example




```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);
}
```






## Use the flag variable in the code


### Define the flag variable


```cpp
#define DEF_bool(name, value, help) _DEFINE_FLAG(bool, name, value, help)
#define DEF_int32(name, value, help) _DEFINE_FLAG(int32, name, value, help)
#define DEF_int64(name, value, help) _DEFINE_FLAG(int64, name, value, help)
#define DEF_uint32(name, value, help) _DEFINE_FLAG(uint32, name, value, help)
#define DEF_uint64(name, value, help) _DEFINE_FLAG(uint64, name, value, help)
#define DEF_double(name, value, help) _DEFINE_FLAG(double, name, value, help)
#define DEF_string(name, value, help) _DEFINE_FLAG(string, name, value, help)
```


- The above 7 macros are used to define 7 different types of flag variables.
- The parameter name is the flag name, the corresponding global variable name is `FLG_name`, the parameter value is the default value, and the parameter help is the comment.
- The flag variable is a global variable and generally should not be defined in the header file.
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
DEF_string(s, "x", "comments"); // fastring FLG_s = "x";
```




### Declare the flag variable


```cpp
#define DEC_bool(name) _DECLARE_FLAG(bool, name)
#define DEC_int32(name) _DECLARE_FLAG(int32, name)
#define DEC_int64(name) _DECLARE_FLAG(int64, name)
#define DEC_uint32(name) _DECLARE_FLAG(uint32, name)
#define DEC_uint64(name) _DECLARE_FLAG(uint64, name)
#define DEC_double(name) _DECLARE_FLAG(double, name)
#define DEC_string(name) _DECLARE_FLAG(string, name)
```


- The 7 macros above are used to declare 7 different types of flag variables.
- The parameter name is the flag name, and the corresponding global variable name is `FLG_name`.
- A flag variable can be defined only once, but it can be declared multiple times, and they can be declared wherever needed.
- The flag variable is generally declared outside the namespace, otherwise it may be not possible to use FLG_name to access the flag variable.



- Example



```cpp
DEC_bool(b);     // extern bool FLG_b;
DEC_int32(i32);  // extern int32 FLG_i32;
DEC_int64(i64);  // extern int64 FLG_i64;
DEC_uint32(u32); // extern uint32 FLG_u32;
DEC_uint64(u64); // extern uint64 FLG_u64;
DEC_double(d);   // extern double FLG_d;
DEC_string(s);   // extern fastring FLG_s;
```




### Use the flag variable


After defining or declaring flag variables, we can use them like ordinary variables:


```cpp
// xx.cc
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






## Use flag in the command line


### Set value of flag variables


Suppose the following flags are defined in the program:


```cpp
DEF_bool(x, false, "bool x");
DEF_bool(y, false, "bool y");
DEF_int32(i, -32, "int32");
DEF_uint64(u, 64, "uint64");
DEF_string(s, "nice", "string");
```


When the program is started, we can modify value of the flag variables through command line parameters:


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

# Integer type flag can pass 8 or hexadecimal number
./xx i=032     # i=26 octal
./xx u=0xff    # u=255 hexadecimal
```




### List Help Information


co/flag supports using `--help` command to print the help information of the program:


```bash
$ ./xx --help
usage:
    ./xx - print flags info
    ./xx --help print this help info
    ./xx --mkconf generate config file
    ./xx --daemon run as a daemon (Linux)
    ./xx xx.conf run with config file
    ./xx config=xx.conf run with config file
    ./xx -x -i=8k -s=ok run with commandline flags
    ./xx -x -i 8k -s ok run with commandline flags
    ./xx x=true i=8192 s=ok run with commandline flags
```




### List all flags


co/flag provides `--` command to list the flags defined in the program:


```bash
$ ./xx --
--config: .path of config file
		type: string default: ""
		from: ../../base/flag.cc
--mkconf: .generate config file
		type: bool default: false
		from: ../../base/flag.cc
```






## Configuration file


### Configuration file format


The configuration file format of co/flag is flexible:


- One configuration item per line, each configuration item corresponds to a flag, and the form is unified as x = y, which looks clear at a glance.
- `#` or `//` means comments, and supports whole-line comments and end-of-line comments.
- The `#` or `//` in quotation marks is not a comment.
- Ignore the blank characters at the beginning or end of the line.
- Blank characters can be added before and after the `=` sign.
- You can use `\` to continue a line to avoid too long a line.
- The string does not support escaping to avoid ambiguity.
- The string can be enclosed in double quotes, single quotes or 3 back quotes.



- Sample configuration file



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




### Generate configuration file


```cpp
DEF_bool(mkconf, false, ".generate config file");
```


- mkconf is a flag defined internally by co/flag, which is a switch for automatically generating configuration file.
- You can use `-mkconf` to generate the configuration file in command line.



```bash
./xx -mkconf            # Generate xx.conf
./xx -mkconf -x u=88    # Custom configuration item value
```




### Adjust the order of configuration items in the configuration file


In the automatically generated configuration file, the configuration items are sorted by flag level, file name, and code line number. If the user wants some configuration items to be ranked higher, the flag level can be set to a smaller value, otherwise, the flag level can be set to a larger value. 


When defining a flag, you can use `#n` at the beginning of the comment to specify the flag level, **n must be an integer between 0 and 99**. If the comment is not empty, there must be a space after n. When not specified, the default flag level is 10.


```cpp
DEF_bool(x, false, "comments");    // The default level is 10
DEF_bool(y, false, "#23");         // The level is 23, and the comment is empty
DEF_bool(z, false, "#3 comments"); // The level is 3
```




### Prohibit configuration items from being generated in the configuration file


Flags beginning with `.`, are **hidden flags**, which will not be present in the configuration file, but can be found with the `--` command in command line. The flag with an empty comment is completely invisible and will neither be generated in the configuration file nor be found with the `--` command.


```cpp
DEF_bool(x, false, ".say something here");
DEF_string(s, "good", "");
```




### Specify the configuration file when the program starts


```cpp
DEF_string(config, "", ".path of config file");
```


- config is a flag defined internally by co/flag, indicating the path of the configuration file.
- You can use `-config` to specify the configuration file in command line.
- Or before calling `flag::init()` in the code, you can modify the value of `FLG_config` to specify the configuration file.



```bash
./xx -config xx.conf

# If the configuration file name ends with .conf or config, 
# and it is the first non-flag parameter on the command line, 
# -config can be omitted.
./xx xx.conf
./xx xx.conf -x
```




## Custom help information


```cpp
DEF_string(help, "", ".help info");
```


- help is a flag defined inside co/flag, which stores the help information of the program. This information can be seen with the command `--help` in command line.
- FLG_help is empty by default, and the default help information provided by co/flag is used.
- When users want to customize the help information, they can modify the value of `FLG_help` before calling `flag::init()`.



- Example



```cpp
#include "co/flag.h"

int main(int argc, char** argv) {
    FLG_help << "usage:\n"
             << "\t./xx -ip 127.0.0.1 -port 7777\n";
    flag::init(argc, argv);
    return 0;
}
```




## Run program as a daemon


```cpp
DEF_bool(daemon, false, "#0 run program as a daemon");
```


- daemon is a flag defined in co/flag. If it is true, the program will run as a daemon. It only supports linux.
- You can use `-daemon` in the command line to specify the program to run in the background as a daemon.



- Example



```bash
./xx -daemon
```
