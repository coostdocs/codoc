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



### Quick start

Run commands below in the root directory of CO to build libco and other projects:

```bash
xmake      # build libco by default
xmake -a   # build all projects (libco, gen, co/test, co/unitest)
```



### Build libco


#### Build static library by default

Run the following commands to build libco (release version of static library by default):

```bash
xmake               # build libco by default
xmake build libco   # build libco
xmake -b libco      # Same as above
xmake -v -b libco   # Same as above, and print detailed compiling information
xmake -vD -b libco  # Same as above, print more detailed compiling information
```


#### Using xmake f command to configure the project

If you want to build dynamic library or set debug mode, you can use `xmake f` command for related configuration. Users can run `xmake f --help` to see the configurable items.

The config file **xmake.conf** can be found in subdirectories of **.xmake** in the root directory of CO.

- Build debug version of dynamic library (for Linux & Mac)

```bash
xmake f -m debug -k shared
xmake -v
```

- Build 32-bit libco for Windows

```bash
xmake f -a x86
xmake -v
```


#### Enable HTTP/SSL features

If you need to use http::Client, SSL or HTTPS features, build libco with the following commands:

```sh
xmake f --with_libcurl=true --with_openssl=true
xmake -a
```

Xmake may install libcurl and openssl from the network, which may be slow.


#### set vs_runtime on Windows

On Windows, CO uses the **MT** runtime library by default, and users can use `xmake f` to configure it:

```cpp
xmake f --vs_runtime=MD
xmake -v
```


#### Android and IOS support

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

[Github Actions](https://github.com/idealvin/co/actions) shows that CO can be built for android and ios (**not tested**).



### Build and run unitest code

[co/unitest](https://github.com/idealvin/co/tree/master/unitest) contains some unit test code, run the following commands to build and run the test program:

```bash
xmake -b unitest       # build unitest
xmake run unitest -a   # run all unit tests
xmake r unitest -a     # same as above
xmake r unitest -os    # run unit test: os
xmake r unitest -json  # run unit test: json
```



### Build and run test code

[co/test](https://github.com/idealvin/co/tree/master/test) contains some test code, add `xx.cc` source file in the co/test directory or its subdirectories, and then run `xmake -b xx` in the root directory of CO to build it.

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

The proto file format can refer to [hello_world.proto](https://github.com/idealvin/co/blob/master/test/so/rpc/hello_world.proto).



### Install libco

After building libco, you can use the `xmake install` command to install libco to the specified directory:

```bash
xmake install -o pkg   # install to pkg
xmake i -o pkg         # same as above
xmake i -o /usr/local  # install to /usr/local
```



### Install libco from xmake repo

```cpp
xrepo install -f "with_openssl=true,with_libcurl=true" co
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
cmake .. -DBUILD_ALL=ON -DCMAKE_INSTALL_PREFIX=pkg
make -j8
make install
```



### Enable HTTP and SSL features

To use HTTP or SSL features, libcurl, zlib, and openssl 1.1.0 or above must be installed.

```sh
mkdir build && cd build
cmake .. -DBUILD_ALL=ON -DWITH_LIBCURL=ON
make -j8
```
