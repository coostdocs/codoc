---
weight: 4
title: "Configuration"
---

## Header

```cpp
#include <co/flag.h>
```

The API is in the `flag` namespace.

## Overview

`flag` provides command-line argument and configuration file parsing functionality. All component configuration items in coost are defined through flags.

- Defining a flag defines a global variable; the variable name is `FLG_<name>`;
- Supports both command-line arguments and configuration files;
- The command line supports `-help` to print help information, `-mkconf` to generate a configuration file, and `-version` to display the program version;

## Defining Flags

### DEF Macros

```cpp
DEF_bool(name, value, help, ...);
DEF_int32(name, value, help, ...);
DEF_int64(name, value, help, ...);
DEF_uint32(name, value, help, ...);
DEF_uint64(name, value, help, ...);
DEF_double(name, value, help, ...);
DEF_string(name, value, help, ...);
```

After definition, access it through a global variable, such as `FLG_name`.

Type correspondence:

| Macro | Type | Internal Identifier |
|---|---|---|
| `DEF_bool` | `bool` | `'b'` |
| `DEF_int32` | `int32` | `'i'` |
| `DEF_int64` | `int64` | `'I'` |
| `DEF_uint32` | `uint32` | `'u'` |
| `DEF_uint64` | `uint64` | `'U'` |
| `DEF_double` | `double` | `'d'` |
| `DEF_string` | `co::string&` | `'s'` |

Example:
```cpp
// Define a global bool variable; the variable name is FLG_debug
DEF_bool(debug, false, "debug mode");
```

### DEC Macros

```cpp
DEC_bool(name);
DEC_int32(name);
DEC_int64(name);
DEC_uint32(name);
DEC_uint64(name);
DEC_double(name);
DEC_string(name);
```

Used for cross-file declarations; the type must match `DEF_xxx`.

### Aliases

Flags support aliases; aliases can be used on the command line or in configuration files.

The last variadic parameter of `DEF_xxx` can specify at most one alias:

```cpp
// d is an alias
DEF_bool(debug, false, "debug mode", d);
```

Aliases are displayed in the help information, for example `-debug,d`.

## Parsing Arguments

```cpp
co::vector<co::string> parse(int argc, char** argv, bool command_line_only=false);
```

- Parses command-line arguments and the configuration file, and updates flag values.
- The return value contains all non-flag arguments;
- When `command_line_only == true`, only command-line arguments are parsed;
- Usually called at the beginning of `main`;
- On error, prints information and exits the program.

## Command-Line Arguments

Two forms are supported:

```bash
-name=value
-name value
```

`-` can be one or more; `-debug` and `--debug` are equivalent.

### bool Type

```bash
-debug          # Equivalent to -debug=true
-debug=true
-debug=false
```

### Integer Units

Integer types support the units `k, m, g, t, p`, case-insensitive, `1k = 1024`.

```bash
-co_stack_size=2m   # 2 * 1024 * 1024
```

### Single-Letter Flag Shorthand Syntax

Single-letter flags support the following two shorthands (command line only):

- Multiple single-letter bool flags can be combined: for example, if x, y, and z are all bool flags, `-xyz` can set all three to true.
- A single-letter integer flag can be written together with its value: for example, `-n8` is equivalent to `-n=8`.

## Configuration File

By default, the first non-flag argument on the command line (whose name must end with `.conf`) is used as the configuration file:

```bash
./xx xx.conf
```

Configuration file format:

```ini
# Comment
debug = true
threads = 8
port = 8080
name = "my app"
n = 8k   # 8192 
```

- `#` indicates a comment;
- Blank lines are supported;
- Leading or trailing spaces on a line are allowed;
- The key is not prefixed with `--`;
- Spaces are allowed before and after `=`;
- If a string has leading or trailing spaces, it needs to be quoted; both single and double quotes are supported;
- Strings support common escape characters; see `co::string::unescape` for details;
- bool supports `true/false` and `1/0`; all other values are treated as `false`;
- Integers support the units `k, m, g, t, p`, case-insensitive;
- If an undefined flag appears in the configuration file, a warning line is printed to the terminal, but the program does not exit;
- Flag names are case-sensitive.

## Command-Line vs. Configuration File Priority

- When both command-line arguments and a configuration file are present, command-line arguments override values in the configuration file;
- A `.conf` passed on the command line overrides the default path set by `set_config_path`.

## Internal Flags

The flag component internally defines three bool flags:

```cpp
DEF_bool(help, false, s_help);
DEF_bool(version, false, s_version);
DEF_bool(mkconf, false, s_mkconf);
```

### -help

Prints help information.

```bash
./xx -help
```

Example help information format:

```text
usage:  ./xx [xx.conf] [-flag [value]] [-flag=value]...

flags:  -name[,alias]  type  comments  (default value)
  -help       b  show help information  (false)
  -version,v  b  show version information  (false)
  -mkconf     b  generate configuration file  (false)

  -boo        b  bool flag  (false)
  ...
```

If the user includes `co/log.h, co/co.h, co/rpc.h`, flags defined internally by coost will also appear in the help information.

### -version

Displays the program version. You need to call `flag::set_program_version` before `flag::parse` to set the version number. If not set, the version information is empty.

```bash
./xx -version
```

### -mkconf

Generates a configuration file:

```bash
./xx -mkconf
```

- Generates the configuration file in the directory where the command is executed;
- File name rule: the executable file name with `.exe` removed, plus `.conf`;
- Contains all user flags as well as flags from the coost components used;
- If you do not want a flag to appear in the configuration file, you can use `flag::hide()` to hide the flag.

## Runtime API

```cpp
// Add an alias; @new_name must have static lifetime
// If @new_name is empty, remove the existing alias
void flag::alias(const char* name, const char* new_name);

// Set the default configuration path
void flag::set_config_path(const char* path);

// Set the program version
void flag::set_program_version(const char* ver);

// Hide a flag so that it does not appear in help information or the configuration file generated by -mkconf
void flag::hide(const char* name);

// Opposite of hide
void flag::unhide(const char* name);

// Set the value of a flag; returns false on error and prints an error message to the terminal
bool flag::set_value(const char* flag_name, const char* value);

// Register a callback to be executed after flag::parse finishes parsing command-line arguments
void flag::run_after_parse(void(*cb)());

// Register a callback to be executed before flag::parse parses command-line arguments
void flag::run_before_parse(void(*cb)());
```

{{< hint warning >}}
The above APIs are not thread-safe and **need to be called before `flag::parse`**.
{{< /hint >}}

- `alias` adds at most one alias.
- After `set_config_path` sets the default configuration path, you do not need to pass a configuration file on the command line; `parse` will parse the configuration file from the default path.
- In `set_value`, `value` is in string form and is parsed internally according to the flag type; an error will not cause the program to exit;

Example:

```cpp
flag::alias("version", "v");
flag::set_value("debug", "true");
```

## Thread Safety

A flag is essentially a global variable or object. If multiple threads access or modify `FLG_xxx`, the user needs to ensure concurrency safety themselves.

## Example

```cpp
#include <co/log.h> // Already includes co/flag.h

DEF_bool(debug, false, "enable debug mode");
DEF_int32(threads, 4, "number of threads");
DEF_uint32(port, 8080, "server port");
DEF_string(name, "coost", "app name", n); // n is an alias

int main(int argc, char** argv) {
    // Set the program version number
    flag::set_program_version("1.0.0");

    // Built-in flag of the logging component; also output logs to the terminal
    flag::set_value("also_log2console", "true");
    
    // Optional: set the default configuration file path
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

Run:

```bash
./app
./app -debug -threads=8 -port 8080 -name=myapp
./app -help
./app -version
./app -mkconf
./app app.conf
```
