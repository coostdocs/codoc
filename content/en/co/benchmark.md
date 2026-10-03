---
weight: 7
title: "Benchmark"
---

## Header

```cpp
#include "co/benchmark.h"
```

## API

There is only one public function, in the `co` namespace:

```cpp
void co::run_benchmarks();
```

- Run benchmarks; results are output in markdown table format.
- `main` is generally written in a fixed way:

```cpp
#include "co/benchmark.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::run_benchmarks();
    return 0;
}
```

## Defining a Benchmark Group

```cpp
BM_group(name) {
    // BM_add(...) { ... }
}
```

- The `BM_group` macro defines a benchmark group. It is actually a function, and users can freely add common initialization code, warm-up code, etc. inside it;
- `name` must be a valid variable name;
- When there are multiple `BM_group`s, `name` must not be duplicated.

## Defining a Benchmark Case

```cpp
BM_add(name) {
    // test code
}
```

- The `BM_add` macro defines a benchmark case. It is actually a code block inside the function defined by `BM_group`;
- `name` is not required to be a valid variable name;

## Subgroups

```cpp
BM_sub_group_begin;
```

- Start a subgroup inside `BM_group`; subgroups are compared independently.
- `BM_add` before the first `BM_sub_group_begin` belongs to the default subgroup.
- Each subgroup uses the first test as the baseline, and the remaining tests calculate `speedup` relative to that baseline.
- The baseline itself displays `speedup` as `-`.

## BM_use

```cpp
BM_use(v);
```

- Prevent the compiler from optimizing away the test code.
- If the test code is optimized away, `iters/s` will be abnormally large; in this case, `BM_use` can be used.
- It can be used for any variable and is usually placed outside `BM_add` to avoid interfering with the code under test.

## Running Logic

- Each `BM_group` internally defines a bool flag with a default value of `false`.
- If the flags of all groups are at their default values, all groups are run.
- If any flag is `true`, only groups whose flag is `true` are run.
- Groups are executed in registration order.

Command-line example:

```bash
./xx            # Run all groups
./xx -rand      # Run only the rand group
./xx -rand -log # Run only the rand and log groups
```

## Example

```cpp
#include "co/benchmark.h"
#include "co/atomic.h"
#include "co/rand.h"
#include <random>

BM_group(atomic) {
    __cacheline_aligned int i = 0;

    BM_add(atomic_inc) {
        co::atomic_inc(&i);
    }
    BM_use(i);

    BM_add(atomic_dec) {
        co::atomic_dec(&i);
    }
    BM_use(i);

    BM_add(atomic_cas) {
        co::atomic_cas(&i, 0, 1);
    }
    BM_use(i);

    BM_add(atomic_or) {
        co::atomic_or(&i, 11);
    }
    BM_use(i);
}

BM_group(rand) {
    // Warm up
    uint32 x = ::rand();
    x += co::rand();

    BM_sub_group_begin;
    BM_add(::rand) {
        x = ::rand();
    }
    BM_use(x);

    BM_add(co::rand) {
        x = co::rand();
    }
    BM_use(x);

    uint32 seed = co::rand();
    BM_add(co::rand(seed)) {
        x = co::rand(seed);
    }
    BM_use(x);

    std::mt19937 m(std::random_device{}());
    BM_add(std::mt19937) {
        x = m();
    }
    BM_use(x);

    uint64 u;
    BM_add(co::rand64) {
        u = co::rand64();
    }
    BM_use(u);

    uint64 seed64 = co::rand64();
    BM_add(co::rand64(seed)) {
        u = co::rand64(seed64);
    }
    BM_use(u);

    BM_sub_group_begin;
    BM_add(co::randstr) {
        (void)co::randstr();
    }

    BM_add(co::randstr(charsets)) {
        (void) co::randstr("0-9a-f", 15);
    }

    char buf[16];
    BM_add(co::randchars) {
        co::randchars(buf, sizeof(buf));
    }
    BM_use(buf);
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::run_benchmarks();
    return 0;
}
```

Run:

```bash
./bm            # Run atomic and rand
./bm -atomic    # Run only atomic
./bm -rand      # Run only rand
```

Test results:

![bm.png](/images/bm.png)

- The output is in Markdown table format.
- `ns/iter`: nanoseconds per iteration.
- `iters/s`: iterations per second.
- `speedup`: speedup factor relative to the test baseline.

## Building and Running coost Internal Benchmarks

The [benchmark](https://github.com/idealvin/coost/tree/master/benchmark) directory contains coost's internal benchmark code. Execute the following commands in the coost root directory to build and run:

```bash
# Build
xmake b benchmark

# Run all benchmark code by default
xmake r benchmark

# Run only the specified benchmarks
xmake r benchmark -rand -mem
```
