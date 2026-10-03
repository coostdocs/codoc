---
weight: 32
title: "Compilation"
---

## Compiler Requirements

**The latest version of coost requires compiler support for C++17**:

- Linux: [gcc](https://gcc.gnu.org/projects/cxx-status.html#cxx17)
- Mac: [clang](https://clang.llvm.org/cxx_status.html)
- Windows: [MSVC](https://visualstudio.microsoft.com/)

## Building with xmake

coost recommends using [xmake](https://github.com/xmake-io/xmake) as the build tool.

### Quick Start

```sh
# All commands are executed in the coost root directory; this will not be repeated below.
xmake       # Build libco by default
xmake -a    # Build all projects (libco, benchmark, gen, test, unitest)
```

In addition, you can use `-v` or `-vD` to make xmake print more detailed compilation information:

```sh
xmake -v -a
```

### Enable the backtrace feature

To print stack traces when a program crashes on Linux and macOS, [libbacktrace](https://github.com/ianlancetaylor/libbacktrace) is required. Newer versions of gcc on Linux have a built-in backtrace library; on macOS it generally needs to be installed manually.

```sh
# First use xmake f to enable the backtrace feature, then compile.
xmake f --with_backtrace=true
xmake -v
```

### Install libco

```sh
xmake install -o pkg          # Package and install to the pkg directory
xmake i -o pkg                # Same as above
xmake install -o /usr/local   # Install to the /usr/local directory
```

### Build and Run coost Test Code

- Unit test code [unitest](https://github.com/idealvin/coost/tree/master/unitest):

```sh
xmake b unitest
xmake r unitest            # Run all unit tests
xmake r unitest -os -json  # Run only os, json
```

- Benchmark test code [benchmark](https://github.com/idealvin/coost/tree/master/benchmark):

```sh
xmake b benchmark
xmake r benchmark             # Run all benchmarks
xmake r benchmark -mem -rand  # Run only mem, rand 
```

- Other test code [test](https://github.com/idealvin/coost/tree/master/test):

```sh
# For xx.cc in the directory, use xmake b xx to compile.
xmake b flag                   # Compile test/flag.cc
xmake b log                    # Compile test/log.cc
xmake b json                   # Compile test/json.cc
xmake b rpc                    # Compile test/rpc.cc

xmake r flag -xz               # Run the flag test program
xmake r log -also_log2console  # Run the log test program
xmake r log -perf              # log performance test
xmake r json                   # Run the json test program
xmake r rpc                    # Start rpc server
xmake r rpc -c                 # Start rpc client
```

### Other Compilation Options

Execute the following command to view supported compilation options:

```sh
xmake f --help
```

### Build the code generation tool gen

```bash
xmake b gen
cp gen /usr/local/bin/
gen hello_world.proto
```

## Building with cmake

### Build libco

```sh
mkdir cmakebuild && cd cmakebuild
cmake ..
make -j8
```

### Build all projects

```sh
mkdir cmakebuild && cd cmakebuild
cmake .. -DBUILD_ALL=ON -DCMAKE_INSTALL_PREFIX=/usr/local
make -j8

cd bin
./unitest  # Run the unit test program
```

### Enable the backtrace feature

```sh
mkdir cmakebuild && cd cmakebuild
cmake .. -DWITH_BACKTRACE=ON -DBUILD_ALL=ON
make -j8
```
