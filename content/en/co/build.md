---
weight: 32
title: "Compiling"
---


## Compiler requirements

The compilers required are as follows:

- Linux: [gcc 4.8+](https://gcc.gnu.org/projects/cxx-status.html#cxx11)
- Mac: [clang 3.3+](https://clang.llvm.org/cxx_status.html)
- Windows: [vs2015+](https://visualstudio.microsoft.com/)




## xmake

CO recommends using [xmake](https://github.com/xmake-io/xmake) as the build tool. 



### Install xmake

For Windows, mac and debian/ubuntu, you can go directly to the [release page of xmake](https://github.com/xmake-io/xmake/releases) to get the installation package. For other systems, please refer to xmake's [Installation instructions](https://xmake.io/#/guide/installation).

Xmake disables compiling as root by default on Linux. [ruki](https://github.com/waruqi) says it is not safe. You can add the following line to `~/.bashrc` to enable it:

```cpp
export XMAKE_ROOT=y
```



### Build

Run commands below in the root directory of CO to build libco and other projects:

```sh
xmake -a   # build all projects (libco, gen, test, unitest)
```

To enable HTTP and SSL features, build with the following commands:

```sh
xmake f --with_libcurl=true --with_openssl=true
xmake -a
```

Xmake may install libcurl and openssl from the network, which may be slow.

`-a` in the command line means to build all projects in CO. If `-a` is not added, only libco will be built by default. In addition, users may use `-v` or `-vD` to print more detailed compiling information:

```sh
xmake -v -a
```


### Compiling options

Xmake provides the `xmake f` command to configure compiling options. Note that **multiple options must be set in a single xmake f command**.


#### Build debug version of libco

```bash
xmake f -m debug
xmake -v
```


#### Build dynamic library

```bash
xmake f -k shared
xmake -v
```

Note that dynamic library is not supported on Windows.


#### Build 32-bit libco

- Windows

```bash
xmake f -a x86
xmake -v
```

- Linux

```bash
xmake f -a i386
xmake -v
```

The `-a` in `xmake f` command means arch. The arch supported by different platforms may be different. Run `xmake f --help` to see the details.


#### set vs_runtime on Windows

On Windows, CO uses the **MT** runtime library by default, and users can use `xmake f` to configure it:

```cpp
xmake f --vs_runtime=MD
xmake -v
```


#### Android and IOS support

CO can also be built on Android and IOS platforms, see [Github Actions](https://github.com/idealvin/coost/actions) for details. Since the author has no front-end development experience, it has not been tested on Android and IOS.

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



### Build and run unitest code

[co/unitest](https://github.com/idealvin/coost/tree/master/unitest) contains some unit test code, run the following commands to build and run the test program:

```bash
xmake -b unitest       # build unitest
xmake r unitest -a     # run all unit tests
xmake r unitest -os    # run unit test: os
xmake r unitest -json  # run unit test: json
```



### Build and run test code

[co/test](https://github.com/idealvin/coost/tree/master/test) contains some test code, add `xx.cc` source file in the co/test directory or its subdirectories, and then run `xmake -b xx` in the root directory of CO to build it.

```bash
xmake -b flag      # compile test/flag.cc
xmake -b log       # compile test/log.cc
xmake -b json      # compile test/json.cc
xmake -b rpc       # compile test/rpc.cc

xmake r flag -xz   # test flag library
xmake r log        # test log library
xmake r log -cout  # also log to terminal
xmake r log -perf  # test performance of log library
xmake r json       # test json
xmake r rpc        # start rpc server
xmake r rpc -c     # start rpc client
```



### Build and use gen

```bash
xmake -b gen
cp gen /usr/local/bin/
gen hello_world.proto
```

The proto file format can refer to [hello_world.proto](https://github.com/idealvin/coost/blob/master/test/so/rpc/hello_world.proto).



### Install libco

After building libco, you can use the `xmake install` command to install libco to the specified directory:

```bash
xmake install -o pkg   # install to pkg
xmake i -o pkg         # same as above
xmake i -o /usr/local  # install to /usr/local
```



### Install libco from xmake repo

```cpp
xrepo install -f "openssl=true,libcurl=true" co
```




## cmake

[izhengfan](https://github.com/izhengfan) helped to provide the cmakefile:

- Only build libco by default.
- The library files are in build/lib directory, and the executable files are in build/bin directory.
- You can use **BUILD_ALL** to build all projects.
- You can use **CMAKE_INSTALL_PREFIX** to specify the installation directory.



### Build libco by default

```bash
mkdir build && cd build
cmake ..
make -j8
```



### Build all projects

```bash
mkdir build && cd build
cmake .. -DBUILD_ALL=ON -DCMAKE_INSTALL_PREFIX=/usr/local
make -j8
make install
```



### Enable HTTP and SSL features

To use HTTP or SSL features, libcurl, zlib, and openssl 1.1.0 or above must be installed.

```sh
mkdir build && cd build
cmake .. -DBUILD_ALL=ON -DWITH_LIBCURL=ON -DWITH_OPENSSL=ON
make -j8
```



### Install libco from vcpkg

```sh
vcpkg install co:x64-windows

# HTTP & SSL support
vcpkg install co[libcurl,openssl]:x64-windows
```
