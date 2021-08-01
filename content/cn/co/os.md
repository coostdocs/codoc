---
weight: 23
title: "操作系统"
---


include: [co/os.h](https://github.com/idealvin/co/blob/master/include/co/os.h).


## os




### os::cpunum
```cpp
int cpunum();
```

- 返回系统 CPU 核数。





### os::cwd
```cpp
fastring cwd();
```

- 返回当前工作目录 (current working directory)。





### os::daemon
```cpp
void daemon();
```

- 将当前进程放到后台运行，仅支持 linux 平台。





### os::env
```cpp
fastring env(const char* name);
```

- 获取系统环境变量的值，参数 name 是环境变量名。





### os::exename
```cpp
fastring exename();
```

- 返回当前进程名，不含路径。





### os::exepath
```cpp
fastring exepath();
```

- 返回当前进程的完整路径。





### os::homedir
```cpp
fastring homedir();
```

- 返回当前用户的 home 目录。





### os::pid
```cpp
int pid();
```

- 返回当前进程的 id。





### os::signal
```cpp
typedef void (*sig_handler_t)(int);
sig_handler_t signal(int sig, sig_handler_t handler, int flag=0);
```

- 设置信号处理函数，参数 sig 是信号值，参数 flag 是 `SA_RESTART`，`SA_ONSTACK` 等选项的组合。
- 参数 flag 仅适用于 linux/mac 平台，windows 平台会忽略此参数。
- 此函数返回旧的信号处理函数。



- 示例
```cpp
void f(int);
os::signal(SIGINT, f);         // user defined handler
os::signal(SIGABRT, SIG_DFL);  // default handler
os::signal(SIGPIPE, SIG_IGN);  // ignore SIGPIPE
```


