---
weight: 32
title: "编译"
---


## 编译器要求

各平台需要安装的编译器如下：

- Linux: [gcc 4.8+](https://gcc.gnu.org/projects/cxx-status.html#cxx11)
- Mac: [clang 3.3+](https://clang.llvm.org/cxx_status.html)
- Windows: [vs2015+](https://visualstudio.microsoft.com/)




## xmake

CO 推荐使用 [xmake](https://github.com/xmake-io/xmake) 作为构建工具。



### 安装 xmake

windows, mac 与 debian/ubuntu 可以直接去 xmake 的 [release](https://github.com/xmake-io/xmake/releases) 页面下载安装包，其他系统请参考 xmake 的 [Installation](https://xmake.io/#/guide/installation) 说明。

xmake 在 linux 上默认禁止 root 用户编译，[ruki](https://github.com/waruqi) 说不安全，可以在 `~/.bashrc` 中加上下面的一行，启用 root 编译:

```cpp
export XMAKE_ROOT=y
```



### 设置 xmake github 镜像代理

xmake 可能会从 github 上拉取三方库，国内 github 下载速度较慢，可以用下面的方式设置镜像代理：

- 安装 xmake 2.5.4 以上版本。
- 执行 `xmake show -l envs` 命令查找 `XMAKE_GLOBALDIR`，在该环境变量指向的目录下面创建一个 **pac.lua** 文件，在文件里加上下面的代码：

```lua
function mirror(url)
    return url:gsub("github.com", "hub.fastgit.org")
end
function main() end
```



### 快速上手

在 CO 根目录执行下述命令构建：

```sh
xmake       # 默认只构建 libco
xmake -a    # 构建所有项目 (libco, gen, co/test, co/unitest)
```



### 构建 libco


#### 默认构建静态库

下面的命令都可以编译 libco，默认编译 release 版本的静态库：

```bash
xmake                   # 默认仅构建 libco
xmake build libco       # 构建 libco
xmake -b libco          # 与上同
xmake -v -b libco       # 与上同, 另外打印详细的编译信息
xmake -vD -b libco      # 与上同, 打印更加详细的编译信息
```


#### 利用 xmake f 命令配置项目

若需要构建动态库、设置 debug 模式等，则可以用 xmake f 命令进行相关配置。用户可以执行 `xmake f --help` 命令查看可配置的项。

需要注意的是**多个配置项必须在一条 xmake f 命令中完成**，未配置的项使用默认值。另外，执行 `xmake f` 命令后，项目根目录下会生成一个 **.xmake** 目录，在该目录的子目录中可以找到 xmake.conf 文件。

- 编译 debug 版本的动态库 (windows 平台不支持动态库)

```bash
xmake f -m debug -k shared
xmake -v
```

- Windows 平台编译 32 位的 libco

```bash
xmake f -a x86
xmake -v
```


#### 启用 HTTP/SSL 特性

若需要使用 http::Client, SSL 或 HTTPS 特性，则可以用下面的命令构建：

```sh
xmake f --with_libcurl=true --with_openssl=true
xmake -a
```

编译过程中，xmake 可能会从网络安装 libcurl 与 openssl，视网络情况，可能会较慢。


#### windows 平台指定 vs_runtime


在 windows 平台，co 默认使用 **MT** 运行库，用户可以使用 xmake f 命令配置 vs_runtime：

```cpp
xmake f --vs_runtime=MD
xmake -v
```


#### Android 与 IOS 支持

- android

```bash
xmake f -p android --ndk=/path/to/android-ndk-r21
xmake -v
```

- ios

```bash
xmake f -p iphoneos
xmake -v
```

[Github Actions](https://github.com/idealvin/co/actions) 显示 CO 在 android 与 ios 平台可以编译通过(**未测试**)。



### 构建及运行 unitest 代码

[co/unitest](https://github.com/idealvin/co/tree/master/unitest) 是单元测试代码，可以执行下述命令编译及运行：

```bash
xmake -b unitest        # build unitest
xmake run unitest -a    # 执行所有单元测试
xmake r unitest -a      # 同上
xmake r unitest -os     # 执行单元测试 os
xmake r unitest -json   # 执行单元测试 json
```



### 构建及运行 test 代码

[co/test](https://github.com/idealvin/co/tree/master/test) 是一些测试代码，在 co/test 目录或其子目录下增加 `xx.cc` 源文件，然后在 co 根目录下执行 `xmake -b xx` 即可构建。

```bash
xmake -b flag                # 编译 test/flag.cc
xmake -b log                 # 编译 test/log.cc
xmake -b json                # 编译 test/json.cc
xmake -b rpc                 # 编译 test/rpc.cc

xmake r flag -xz             # 测试 flag 库
xmake r log                  # 测试 log 库
xmake r log -cout            # 终端也打印日志
xmake r log -perf            # log 库性能测试
xmake r json                 # 测试 json
xmake r rpc                  # 启动 rpc server
xmake r rpc -c               # 启动 rpc client
```



### 构建及使用 gen

```bash
xmake -b gen
cp gen /usr/local/bin/
gen hello_world.proto
```

proto 文件格式可以参考 [hello_world.proto](https://github.com/idealvin/co/blob/master/test/so/rpc/hello_world.proto)。



### 安装 libco

构建完 libco 后，可以用 `xmake install` 命令安装 libco 到指定的目录：

```bash
xmake install -o pkg          # 打包安装到 pkg 目录
xmake i -o pkg                # 同上
xmake i -o /usr/local         # 安装到 /usr/local 目录
```



### 从 xmake repo 安装 libco

```cpp
xrepo install -f "with_openssl=true,with_libcurl=true" co
```




## cmake

[izhengfan](https://github.com/izhengfan) 帮忙提供了 cmake 编译脚本：

- 默认只编译 libco。
- 编译生成的库文件在 build/lib 目录下，可执行文件在 build/bin 目录下。
- 可以用 **BUILD_ALL** 指定编译所有项目。
- 可以用 **CMAKE_INSTALL_PREFIX** 指定安装目录。
- cmake 只提供简单的编译选项，若需要更复杂的配置，请使用 xmake。



### 默认构建 libco

```bash
mkdir build && cd build
cmake ..
make -j8
```



### 构建所有项目

```bash
mkdir build && cd build
cmake .. -DBUILD_ALL=ON -DCMAKE_INSTALL_PREFIX=pkg
make -j8
```



### 启用 HTTP 与 SSL 特性 

使用 HTTP 或 SSL 特性，需要安装 libcurl, zlib, 以及 openssl 1.1.0 或以上版本。

```sh
mkdir build && cd build
cmake .. -DBUILD_ALL=ON -DWITH_LIBCURL=ON
make -j8
```
