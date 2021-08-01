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


co recommends using [xmake](https://github.com/xmake-io/xmake) as the build tool. 




### Install xmake


For windows, mac and debian/ubuntu, you can go directly to the [release page of xmake](https://github.com/xmake-io/xmake/releases) to get the installation package. For other systems, please refer to xmake's [Installation instructions](https://xmake.io/#/guide/installation).


Xmake disables compiling as root by default on linux. [ruki](https://github.com/waruqi) says it is not safe. You can add the following line to `~/.bashrc` to enable it:

```cpp
export XMAKE_ROOT=y
```




### Quick start


```bash
# All commands are executed in the root directory of co (the same below)
xmake      # build libco and gen by default
xmake -a   # build all projects (libco, gen, co/test, co/unitest)
```




### Build libco


```bash
xmake build libco     # build libco only
xmake -b libco        # same as above
```




### Build and run unitest code


[co/unitest](https://github.com/idealvin/co/tree/master/unitest) contains some unit test codes, which are used to check the correctness of the functionality of libco.


```bash
xmake build unitest    # build can be abbreviated as -b
xmake run unitest -a   # run all unit tests
xmake r unitest -a     # same as above
xmake r unitest -os    # run unit test os
xmake r unitest -json  # run unit test json
```




### Build and run test code


[co/test](https://github.com/idealvin/co/tree/master/test) contains some test codes. You can easily add a source file like `xx.cc` in the directory `co/test` or its subdirectories, and then run `xmake -b xx` to build it.
```bash
xmake build flag             # flag.cc
xmake build log              # log.cc
xmake build json             # json.cc
xmake build rpc              # rpc.cc
xmake build easy             # so/easy.cc

xmake r flag -xz             # test flag
xmake r log                  # test log
xmake r log -cout            # also log to terminal
xmake r log -perf            # performance test
xmake r json                 # test json
xmake r rpc                  # start rpc server
xmake r rpc -c               # start rpc client
xmake r easy -d xxx          # start web server
```




### Build gen


```bash
# It is recommended to put gen in the system directory (e.g. /usr/local/bin/).
xmake -b gen
gen hello_world.proto
```


Proto file format can refer to [hello_world.proto](https://github.com/idealvin/co/blob/master/test/__/rpc/hello_world.proto).


### Installation


```bash
# Install header files, libco, gen by default.
xmake install -o pkg         # package related files to the pkg directory
xmake i -o pkg               # the same as above
xmake install -o /usr/local  # install to the /usr/local directory
```






## cmake


[izhengfan](https://github.com/izhengfan) helped provide cmake support:


- By default, only libco and gen are build.

- The library files are in the build/lib directory, and the executable files are in the build/bin directory.

- You can use **BUILD_ALL** to compile all projects.

- You can use **CMAKE_INSTALL_PREFIX** to specify the installation directory.




```bash
# build libco & gen by default
mkdir build && cd build
cmake ..
make -j8
make install

# build all projects and install to pkg
mkdir build && cd build
cmake .. -DBUILD_ALL=ON -DCMAKE_INSTALL_PREFIX=pkg
make -j8
make install
```
