---
weight: 32
title: "编译"
---


## 编译器要求

**最新版本 coost 需要编译器支持 C++17**：

- Linux: [gcc](https://gcc.gnu.org/projects/cxx-status.html#cxx17)
- Mac: [clang](https://clang.llvm.org/cxx_status.html)
- Windows: [MSVC](https://visualstudio.microsoft.com/)

## 用 xmake 构建

coost 推荐使用 [xmake](https://github.com/xmake-io/xmake) 作为构建工具。


### 快速上手

```sh
# 所有命令都在 coost 根目录执行，后面不再说明
xmake       # 默认构建 libco
xmake -a    # 构建所有项目 (libco, benchmark, gen, test, unitest)
```

另外，可以用 `-v` 或 `-vD` 让 xmake 打印更详细的编译信息：

```sh
xmake -v -a
```


### 启用 backtrace 特性

在 Linux、macOS 上打印程序崩溃的堆栈信息，需要 [libbacktrace](https://github.com/ianlancetaylor/libbacktrace)，Linux 上较新版本 gcc 已内置 backtrace 库，macOS 上一般需要手动安装。

```sh
# 先用 xmake f 启用 backtrace 特性，再编译
xmake f --with_backtrace=true
xmake -v
```


### 安装 libco

```sh
xmake install -o pkg          # 打包安装到 pkg 目录
xmake i -o pkg                # 同上
xmake install -o /usr/local   # 安装到 /usr/local 目录
```


### 编译及运行 coost 测试代码

- 单元测试代码 [unitest](https://github.com/idealvin/coost/tree/master/unitest):

```sh
xmake b unitest
xmake r unitest            # 运行所有单元测试
xmake r unitest -os -json  # 仅运行 os, json
```

- 性能基准测试代码 [benchmark](https://github.com/idealvin/coost/tree/master/benchmark):

```sh
xmake b benchmark
xmake r benchmark             # 运行所有基准测试
xmake r benchmark -mem -rand  # 仅运行 mem, rand 
```

- 其他测试代码 [test](https://github.com/idealvin/coost/tree/master/test):

```sh
# 目录下 xx.cc 用 xmake b xx 编译
xmake b flag                   # 编译 test/flag.cc
xmake b log                    # 编译 test/log.cc
xmake b json                   # 编译 test/json.cc
xmake b rpc                    # 编译 test/rpc.cc

xmake r flag -xz               # 运行 flag 测试程序
xmake r log -also_log2console  # 运行 log 测试程序
xmake r log -perf              # log 性能测试
xmake r json                   # 运行 json 测试程序
xmake r rpc                    # 启动 rpc server
xmake r rpc -c                 # 启动 rpc client
```


### 其他编译选项

执行如下命令查看支持的编译选项：

```sh
xmake f --help
```


### 构建代码生成工具 gen

```bash
xmake b gen
cp gen /usr/local/bin/
gen hello_world.proto
```



## 用 cmake 构建

### 构建 libco

```sh
mkdir cmakebuild && cd cmakebuild
cmake ..
make -j8
```

### 构建所有项目

```sh
mkdir cmakebuild && cd cmakebuild
cmake .. -DBUILD_ALL=ON -DCMAKE_INSTALL_PREFIX=/usr/local
make -j8

cd bin
./unitest  # 运行单元测试程序
```

### 启用 backtrace 特性

```sh
mkdir cmakebuild && cd cmakebuild
cmake .. -DWITH_BACKTRACE=ON -DBUILD_ALL=ON
make -j8
```
