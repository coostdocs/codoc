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


co 推荐使用 [xmake](https://github.com/xmake-io/xmake) 作为构建工具。




### 安装 xmake


windows, mac 与 debian/ubuntu 可以直接去 xmake 的 [release](https://github.com/xmake-io/xmake/releases) 页面下载安装包，其他系统请参考 xmake 的 [Installation](https://xmake.io/#/guide/installation) 说明。


xmake 在 linux 上默认禁止 root 用户编译，[ruki](https://github.com/waruqi) 说不安全，可以在 `~/.bashrc` 中加上下面的一行，启用 root 编译:
```cpp
export XMAKE_ROOT=y
```




### 设置 xmake github 镜像代理


co 中的 HTTP/SSL 特性依赖于 libcurl 与 openssl，启用 HTTP/SSL 特性时，xmake 会自动安装所需要的三方库。xmake 可能从 github 上拉取三方库，国内 github 下载速度较慢，建议用下面的方式设置 github 镜像代理：


- 安装 xmake 2.5.4 以上版本。
- 执行 `xmake show -l envs` 命令查找 `XMAKE_GLOBALDIR` 环境变量的值，在该环境变量指向的目录下面创建一个 **pac.lua** 文件，在文件里加上下面的代码：
```lua
function mirror(url)
     return url:gsub("github.com", "hub.fastgit.org")
end
function main() end
```




### 快速上手
```bash
# 所有命令都在 co 根目录执行，后面不再说明
xmake       # 默认构建 libco
xmake -a    # 构建所有项目 (libco, gen, co/test, co/unitest)
```




### 构建 libco


#### 默认构建静态库
```bash
xmake build libco       # 仅构建 libco
xmake -b libco          # 与上同
xmake -v -b libco       # 与上同, 另外打印详细的编译信息
xmake -vD -b libco      # 与上同, 打印更加详细的编译信息
```

- 默认构建 release 版本的静态库。



#### 利用 xmake f 命令配置项目


若需要构建动态库、设置 debug 模式等，则可以用 xmake f 命令进行相关配置。用户可以执行 `**xmake f --help**` 命令查看可配置的项。
```bash
# 编译 debug 版本的动态库
xmake f -m debug -k shared
xmake -v

# Windows 编译x86的版本
xmake f -a x86 
xmake -a
# 查看帮助
xmake f --help
```

- 注意**多个配置项必须在一条 xmake f 命令中完成**，未配置的项使用默认值。
- 执行 xmake 配置命令后，在项目根目录下会生成一个 **.xmake** 目录，在该目录的子目录中可以找到 xmake.conf 文件。
- windows 平台不支持动态库。



#### 启用 HTTP/SSL 特性


co 中的 http::Client 基于 libcurl 实现，co 中的 SSL 特性基于 openssl 实现。
```bash
xmake f --with_libcurl=true --with_openssl=true
xmake -a
```

- 此过程中，若 xmake 未检测到 libcurl, openssl，则会自动安装这些三方依赖库。



#### windows 平台指定 vs_runtime


在 windows 平台，co 默认使用 **MT** 运行库，用户可以使用 xmake f 命令配置 vs_runtime：
```cpp
xmake f --vs_runtime=MD
xmake -v
```


#### Android 与 IOS 支持
```bash
# android
xmake f -p android --ndk=/path/to/android-ndk-r21
xmake -v

# ios
xmake f -p iphoneos
xmake -v
```

- github ci 显示 co 在 android 与 ios 平台可以编译通过(**未测试**)。





### 构建及运行 unitest 代码


[co/unitest](https://github.com/idealvin/co/tree/master/unitest) 是单元测试代码，用于检验 libco 库功能的正确性。
```bash
xmake build unitest     # build 可以简写为 -b
xmake run unitest -a    # 执行所有单元测试
xmake r unitest -a      # 同上
xmake r unitest -os     # 执行单元测试 os
xmake r unitest -json   # 执行单元测试 json
```




### 构建及运行 test 代码


[co/test](https://github.com/idealvin/co/tree/master/test) 是一些测试代码。在 co/test 目录或子目录下增加 `xx.cc` 源文件，然后在 co 根目录下执行 `xmake -b xx` 即可构建。
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




### 构建 gen
```bash
# 建议将 gen 放到系统目录下(如 /usr/local/bin/).
xmake -b gen
gen hello_world.proto
```
proto 文件格式可以参考 [hello_world.proto](https://github.com/idealvin/co/blob/master/test/__/rpc/hello_world.proto)。




### 安装 libco
```bash
# 默认安装头文件与 libco
xmake install -o pkg          # 打包安装到 pkg 目录
xmake i -o pkg                # 同上
xmake i -o /usr/local         # 安装到 /usr/local 目录
```




### 从 xmake repo 安装 libco
```cpp
xrepo install -f "with_openssl=true,with_libcurl=true" co
```


## cmake


[izhengfan](https://github.com/izhengfan) 帮忙提供了 cmake 支持：

- 默认只编译 libco。
- 编译生成的库文件在 build/lib 目录下，可执行文件在 build/bin 目录下。
- 可以用 **BUILD_ALL** 指定编译所有项目。
- 可以用** CMAKE_INSTALL_PREFIX **指定安装目录。
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
make install
```


