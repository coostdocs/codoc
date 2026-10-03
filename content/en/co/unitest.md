---
weight: 6
title: "Unit Test"
---

## Header

```cpp
#include "co/unitest.h"
```

## API

There is only one public function, in the `co` namespace:

```cpp
int co::run_unitests();
```

- Runs unit tests and returns the number of failed test cases;
- `main` is generally written in a fixed way:

```cpp
#include "co/unitest.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::run_unitests();
    return 0;
}
```

## Defining a Test Unit

```cpp
DEF_test(name) {
    // test code
    // DEF_case(xxx) {}
}
```

- The `DEF_test` macro defines a test unit. It is actually a function, and users can freely add code inside it, such as initialization code shared by test cases;
- `name` must be a valid variable name;
- When there are multiple `DEF_test`s, `name` must not be duplicated.

## Defining a Test Case

```cpp
DEF_case(name) {
    // test case code
}
```

- The `DEF_case` macro defines a test case. It is actually a code block inside the function defined by `DEF_test`;
- `name` is not required to be a valid variable name;
- If `DEF_test` does not contain any `DEF_case`, a default test case is created.

## EXPECT Macros

```cpp
EXPECT(x)
EXPECT_EQ(x, y)   // ==
EXPECT_NE(x, y)   // !=
EXPECT_GE(x, y)   // >=
EXPECT_LE(x, y)   // <=
EXPECT_GT(x, y)   // >
EXPECT_LT(x, y)   // <
```

- Failure messages are summarized at the end.
- The arguments in `EXPECT_XX` need to support the corresponding comparison operators and be printable, i.e. support:
  ```cpp
  operator<<(co::string&, const T&);
  ```  

## Running Logic

- Each `DEF_test` internally defines a bool flag with a default value of `false`.
- If the flags of all test units are at their default values, all test units are run.
- If any flag is `true`, only test units whose flag is `true` are run.
- Test units are executed in registration order.

## Example

```cpp
#include "co/unitest.h"

DEF_test(math) {
    DEF_case(add) {
        EXPECT_EQ(1 + 1, 2);
        EXPECT_NE(1 + 1, 3);
        EXPECT_GT(2, 1);
        EXPECT_LT(1, 2);
        EXPECT_GE(2, 2);
        EXPECT_LE(2, 2);
        EXPECT(1 + 1 == 2);
    }

    DEF_case(mul) {
        EXPECT_EQ(2 * 3, 6);
        EXPECT_NE(2 * 3, 5);
    }
}

DEF_test(string) {
    co::string s;

    // Not in DEF_case; default test case
    EXPECT(s.empty());

    DEF_case(empty) {
        EXPECT(s.empty());
    }

    DEF_case(size) {
        s = "hello";
        EXPECT_EQ(s.size(), 5);
    }
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::run_unitests();
    return 0;
}
```

Run:

```bash
./xx            # Run both math and string units
./xx -math      # Run only math
./xx -string    # Run only string
```

## Building and Running coost Internal Unit Tests

The [unitest](https://github.com/idealvin/coost/tree/master/unitest) directory contains coost's internal unit test code. Execute the following commands in the coost root directory to build and run:

```bash
# Build
xmake b unitest

# Run all unit test code by default
xmake r unitest

# Run only the specified test units
xmake r unitest -os -json
```

Test result examples:

- All tests passed

![unitest_passed.png](/images/unitest_passed.png)

- A test case failed

![unitest_failed.png](/images/unitest_failed.png)
